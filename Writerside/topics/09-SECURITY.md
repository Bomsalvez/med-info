# Seguranca e LGPD - MedInfo

## 1. Visao Geral

Este documento descreve as medidas de seguranca implementadas no MedInfo e a conformidade com a Lei Geral de Protecao de Dados (LGPD - Lei 13.709/2018).

---

## 2. Classificacao de Dados

### 2.1 Tipos de Dados Coletados

| Categoria | Dados | Sensibilidade | Base Legal LGPD |
|-----------|-------|---------------|-----------------|
| Cadastrais | Nome, email | Pessoal | Execucao de contrato |
| Autenticacao | Senha (hash) | Pessoal | Execucao de contrato |
| Profissionais | CRM, CRF, COREN | Pessoal | Consentimento |
| Academicos | Matricula, instituicao | Pessoal | Consentimento |
| Uso | Buscas, favoritos | Pessoal | Interesse legitimo |
| Pagamento | Dados de transacao | Pessoal | Execucao de contrato |
| Dispositivo | Device ID, IP | Pessoal | Interesse legitimo |

### 2.2 Dados NAO Coletados

| Dado | Motivo |
|------|--------|
| CPF | Nao necessario para o servico |
| Dados de saude do usuario | Fora do escopo |
| Localizacao precisa | Nao necessario |
| Contatos do dispositivo | Nao necessario |
| Historico de navegacao | Fora do escopo |

---

## 3. Autenticacao e Autorizacao

### 3.1 Autenticacao

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUXO DE AUTENTICACAO                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Usuario envia credenciais (email + senha)               │
│                     │                                        │
│                     ▼                                        │
│  2. Backend valida credenciais                              │
│     - Busca usuario por email                               │
│     - Compara hash bcrypt (cost factor 12)                  │
│                     │                                        │
│                     ▼                                        │
│  3. Gera tokens JWT                                         │
│     - Access Token (7 dias)                                 │
│     - Refresh Token (30 dias)                               │
│                     │                                        │
│                     ▼                                        │
│  4. Armazena refresh token (hash) no Redis                  │
│                     │                                        │
│                     ▼                                        │
│  5. Retorna tokens ao cliente                               │
│     - Mobile armazena no Android Keystore                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Especificacoes JWT

| Parametro | Valor | Descricao |
|-----------|-------|-----------|
| Algoritmo | HS256 | HMAC SHA-256 |
| Access Token TTL | 7 dias | Tempo de vida |
| Refresh Token TTL | 30 dias | Tempo de renovacao |
| Issuer | medinfo.com.br | Emissor |
| Audience | medinfo-mobile | Destinatario |

#### Payload do Token

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@email.com",
  "role": "PROFESSIONAL",
  "isValidated": true,
  "iat": 1705312200,
  "exp": 1705917000,
  "iss": "medinfo.com.br",
  "aud": "medinfo-mobile"
}
```

### 3.3 Autorizacao (RBAC)

| Recurso | GENERAL | STUDENT | PROFESSIONAL |
|---------|---------|---------|--------------|
| Bula Simplificada | ✅ | ✅ | ✅ |
| Bula Tecnica | ❌ | ✅* | ✅* |
| Calculadora Dosagem | ❌ | ✅* | ✅* |
| Favoritos | ✅ | ✅ | ✅ |
| Consulta Online | ✅ | ✅ | ✅ |

*Requer `isValidated = true`

### 3.4 Implementacao NestJS

```typescript
// JWT Strategy
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: JwtPayload): Promise<UserPayload> {
    return {
      userId: payload.sub,
      email: payload.email,
      role: payload.role,
      isValidated: payload.isValidated,
    };
  }
}

// Role Guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.role === role);
  }
}

// Validated Guard
@Injectable()
export class ValidatedGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const { user } = context.switchToHttp().getRequest();
    return user.isValidated === true;
  }
}

// Decorator de uso
@Controller('dosage')
@UseGuards(JwtAuthGuard, RolesGuard, ValidatedGuard)
@Roles(UserRole.PROFESSIONAL, UserRole.STUDENT)
export class DosageController {
  @Post('calculate')
  calculate(@Body() dto: CalculateDosageDto) {
    // Apenas usuarios validados chegam aqui
  }
}
```

---

## 4. Criptografia

### 4.1 Em Transito

| Camada | Protocolo | Configuracao |
|--------|-----------|--------------|
| API REST | TLS 1.3 | Certificado Let's Encrypt |
| WebSocket | WSS | TLS 1.3 |

### 4.2 Em Repouso

| Dado | Metodo | Chave |
|------|--------|-------|
| Senha | bcrypt | cost factor 12 |
| Refresh Token | SHA-256 | - |
| Dados sensiveis DB | AES-256-GCM | KMS |
| Backup | AES-256 | KMS |

### 4.3 No Dispositivo (Android)

```kotlin
// Armazenamento seguro de tokens
object SecureStorage {
    private const val KEYSTORE_ALIAS = "medinfo_key"

    fun saveToken(context: Context, token: String) {
        val encryptedPrefs = EncryptedSharedPreferences.create(
            context,
            "medinfo_secure_prefs",
            getMasterKey(context),
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
        encryptedPrefs.edit().putString("access_token", token).apply()
    }

    private fun getMasterKey(context: Context): MasterKey {
        return MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build()
    }
}

// Room com criptografia
val passphrase = SQLiteDatabase.getBytes("medinfo_secure".toCharArray())
val factory = SupportFactory(passphrase)

Room.databaseBuilder(context, MedInfoDatabase::class.java, "medinfo.db")
    .openHelperFactory(factory)
    .build()
```

---

## 5. Protecao contra Ataques

### 5.1 OWASP Top 10

| Vulnerabilidade | Mitigacao |
|-----------------|-----------|
| Injection | Prepared statements, ORM, input validation |
| Broken Auth | JWT com rotacao, rate limiting |
| Sensitive Data Exposure | TLS, criptografia, masking em logs |
| XXE | Desabilitado no parser XML |
| Broken Access Control | RBAC, guards, validacao server-side |
| Security Misconfiguration | Headers de seguranca, config review |
| XSS | CSP, sanitizacao, escape output |
| Insecure Deserialization | class-transformer com whitelist |
| Using Components with Vulns | Dependabot, auditorias regulares |
| Insufficient Logging | Audit trail, alertas |

### 5.2 Rate Limiting

```typescript
// NestJS Throttler
@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'short',
        ttl: 1000,  // 1 segundo
        limit: 3,   // 3 requests
      },
      {
        name: 'medium',
        ttl: 10000, // 10 segundos
        limit: 20,  // 20 requests
      },
      {
        name: 'long',
        ttl: 60000, // 1 minuto
        limit: 100, // 100 requests
      },
    ]),
  ],
})
export class AppModule {}

// Aplicacao especifica
@Controller('auth')
export class AuthController {
  @Post('login')
  @Throttle({ short: { limit: 5, ttl: 60000 } }) // 5 tentativas por minuto
  async login(@Body() dto: LoginDto) {}
}
```

### 5.3 Headers de Seguranca

```typescript
// Helmet middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));

// CORS
app.enableCors({
  origin: ['https://medinfo.com.br'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  credentials: true,
});
```

---

## 6. LGPD - Conformidade

### 6.1 Direitos do Titular

| Direito | Implementacao |
|---------|---------------|
| Confirmacao | GET /users/me |
| Acesso | GET /users/me/data (exportacao completa) |
| Correcao | PUT /users/me |
| Anonimizacao | DELETE /users/me (soft delete + anonimizacao) |
| Portabilidade | GET /users/me/export (JSON/CSV) |
| Eliminacao | DELETE /users/me/permanent |
| Revogacao | PUT /users/me/consent |

### 6.2 Consentimento

```typescript
// Registro de consentimento
interface Consent {
  userId: string;
  type: ConsentType;
  granted: boolean;
  grantedAt: Date;
  revokedAt?: Date;
  version: string;
  ipAddress: string;
}

enum ConsentType {
  TERMS_OF_SERVICE = 'TERMS_OF_SERVICE',
  PRIVACY_POLICY = 'PRIVACY_POLICY',
  MARKETING = 'MARKETING',
  PROFESSIONAL_VALIDATION = 'PROFESSIONAL_VALIDATION',
}
```

### 6.3 Termos e Politicas

| Documento | Versao | Obrigatorio |
|-----------|--------|-------------|
| Termos de Uso | 1.0 | Sim |
| Politica de Privacidade | 1.0 | Sim |
| Politica de Cookies | 1.0 | Nao (mobile) |

### 6.4 Retencao de Dados

| Dado | Retencao | Apos Exclusao |
|------|----------|---------------|
| Dados cadastrais | Enquanto ativo | Anonimizacao em 30 dias |
| Logs de acesso | 6 meses | Exclusao automatica |
| Dados de pagamento | 5 anos (fiscal) | Mantido anonimizado |
| Validacao profissional | Enquanto ativo | Anonimizacao em 30 dias |
| Buscas/Favoritos | Enquanto ativo | Exclusao imediata |

### 6.5 Anonimizacao

```typescript
// Processo de anonimizacao
async anonymizeUser(userId: string): Promise<void> {
  const user = await this.userRepository.findById(userId);

  // Anonimizar dados pessoais
  const anonymizedUser = {
    ...user,
    email: `deleted_${uuid()}@anonymous.medinfo.local`,
    name: 'Usuario Anonimizado',
    passwordHash: null,
    isDeleted: true,
    deletedAt: new Date(),
  };

  // Remover validacao profissional
  await this.validationRepository.deleteByUserId(userId);

  // Manter pagamentos com dados anonimizados (requisito fiscal)
  await this.paymentRepository.anonymizeByUserId(userId);

  // Remover favoritos
  await this.favoriteRepository.deleteByUserId(userId);

  await this.userRepository.save(anonymizedUser);
}
```

---

## 7. Auditoria e Logging

### 7.1 Eventos Auditados

| Evento | Dados Registrados |
|--------|-------------------|
| Login | userId, ip, userAgent, timestamp, success |
| Logout | userId, timestamp |
| Registro | email (masked), timestamp |
| Validacao | userId, councilType, status, timestamp |
| Pagamento | userId, amount, status, timestamp |
| Acesso a dados | userId, resource, action, timestamp |
| Exclusao de conta | userId, timestamp, reason |

### 7.2 Formato de Log

```json
{
  "timestamp": "2026-01-15T10:30:00Z",
  "level": "info",
  "event": "USER_LOGIN",
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "ip": "192.168.1.100",
  "userAgent": "MedInfo/1.0.0 (Android 14)",
  "success": true,
  "duration": 150,
  "requestId": "req_abc123"
}
```

### 7.3 Mascaramento

```typescript
// Dados sensiveis em logs
const maskEmail = (email: string): string => {
  const [local, domain] = email.split('@');
  return `${local.substring(0, 2)}***@${domain}`;
};

// Exemplo: joao.silva@email.com -> jo***@email.com
```

---

## 8. Incidentes de Seguranca

### 8.1 Processo de Resposta

```
1. DETECCAO
   - Alertas automaticos
   - Denuncia de usuario
   - Auditoria interna

2. CONTENCAO
   - Isolar sistema afetado
   - Bloquear acesso suspeito
   - Preservar evidencias

3. ERRADICACAO
   - Identificar causa raiz
   - Corrigir vulnerabilidade
   - Atualizar sistemas

4. RECUPERACAO
   - Restaurar servicos
   - Monitorar anomalias
   - Comunicar stakeholders

5. LICOES APRENDIDAS
   - Documentar incidente
   - Atualizar processos
   - Treinar equipe
```

### 8.2 Notificacao ANPD

Em caso de incidente com dados pessoais:

1. Avaliar risco aos titulares
2. Notificar ANPD em ate 2 dias uteis (se risco alto)
3. Comunicar titulares afetados
4. Documentar acoes tomadas

---

## 9. Checklist de Seguranca

### 9.1 Desenvolvimento

- [ ] Code review obrigatorio
- [ ] Analise estatica (ESLint, SonarQube)
- [ ] Testes de seguranca automatizados
- [ ] Dependencias atualizadas
- [ ] Secrets em vault (nao em codigo)

### 9.2 Deploy

- [ ] HTTPS obrigatorio
- [ ] Headers de seguranca configurados
- [ ] Rate limiting ativo
- [ ] Logs de auditoria ativos
- [ ] Backups criptografados

### 9.3 Operacao

- [ ] Monitoramento de anomalias
- [ ] Alertas de seguranca configurados
- [ ] Plano de resposta a incidentes
- [ ] Treinamento da equipe
- [ ] Auditorias periodicas

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
