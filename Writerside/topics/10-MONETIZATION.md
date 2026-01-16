# Modelo de Monetizacao - MedInfo

## 1. Visao Geral

O MedInfo adota um modelo **Freemium**, oferecendo funcionalidades essenciais gratuitamente e cobrando por consultas expandidas a medicamentos nao disponiveis no banco de dados local.

---

## 2. Modelo de Negocios

### 2.1 Canvas Resumido

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MODELO DE NEGOCIOS                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  PROPOSTA DE VALOR                                                       │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ • Acesso gratuito a 500+ medicamentos comuns (offline)             │ │
│  │ • Bula adaptativa (tecnica/simplificada)                           │ │
│  │ • Calculadora de dosagem para profissionais                        │ │
│  │ • Consultas expandidas sob demanda (pago)                          │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  SEGMENTOS DE CLIENTES                                                   │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐│
│  │ Profissionais   │ │ Estudantes      │ │ Publico Geral              ││
│  │ de Saude        │ │ da Area         │ │                            ││
│  │ (Pagantes)      │ │ (Pagantes)      │ │ (Majoritariamente Free)    ││
│  └─────────────────┘ └─────────────────┘ └─────────────────────────────┘│
│                                                                          │
│  FONTES DE RECEITA                                                       │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ • Consultas online avulsas (pay-per-use)                          │ │
│  │ • Pacotes de consultas (pre-pagos com desconto)                   │ │
│  │ • [Futuro] Assinatura premium                                     │ │
│  │ • [Futuro] B2B para clinicas/hospitais                            │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Estrutura de Precos

### 3.1 Plano Gratuito

| Funcionalidade | Limite | Custo |
|----------------|--------|-------|
| Medicamentos offline | 500+ | R$ 0,00 |
| Bula simplificada | Ilimitado | R$ 0,00 |
| Bula tecnica (validados) | Ilimitado | R$ 0,00 |
| Calculadora dosagem | Ilimitado | R$ 0,00 |
| Favoritos | 50 | R$ 0,00 |
| Busca por codigo barras | Sim | R$ 0,00 |

### 3.2 Consultas Online (Pay-per-Use)

| Tipo | Preco | Descricao |
|------|-------|-----------|
| Consulta avulsa | R$ 2,99 | 1 medicamento nao disponivel offline |
| Pacote 10 | R$ 24,90 | 10 consultas (17% desconto) |
| Pacote 25 | R$ 54,90 | 25 consultas (27% desconto) |
| Pacote 50 | R$ 99,90 | 50 consultas (33% desconto) |

### 3.3 [Futuro] Plano Premium

| Funcionalidade | Mensal | Anual |
|----------------|--------|-------|
| Consultas ilimitadas | R$ 19,90 | R$ 199,00 (17% desc) |
| Alertas de interacao | Incluso | Incluso |
| Exportacao de dados | Incluso | Incluso |
| Suporte prioritario | Incluso | Incluso |

---

## 4. Fluxo de Monetizacao

### 4.1 Jornada do Usuario

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    JORNADA DE CONVERSAO                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  AQUISICAO                                                               │
│  ┌────────────────┐                                                     │
│  │ Download do    │                                                     │
│  │ app (gratuito) │                                                     │
│  └───────┬────────┘                                                     │
│          │                                                               │
│          ▼                                                               │
│  ATIVACAO                                                                │
│  ┌────────────────┐                                                     │
│  │ Cadastro       │────▶ Validacao (se profissional/estudante)         │
│  │ (gratuito)     │                                                     │
│  └───────┬────────┘                                                     │
│          │                                                               │
│          ▼                                                               │
│  USO GRATUITO                                                            │
│  ┌────────────────┐                                                     │
│  │ Consultas      │                                                     │
│  │ offline        │                                                     │
│  │ (500+ meds)    │                                                     │
│  └───────┬────────┘                                                     │
│          │                                                               │
│          ▼                                                               │
│  GATILHO DE CONVERSAO                                                    │
│  ┌────────────────┐                                                     │
│  │ Busca por      │                                                     │
│  │ medicamento    │                                                     │
│  │ NAO disponivel │                                                     │
│  │ offline        │                                                     │
│  └───────┬────────┘                                                     │
│          │                                                               │
│          ▼                                                               │
│  CONVERSAO                                                               │
│  ┌────────────────┐     ┌────────────────┐     ┌────────────────┐      │
│  │ Consulta       │ ou  │ Pacote de      │ ou  │ [Futuro]       │      │
│  │ avulsa         │     │ consultas      │     │ Assinatura     │      │
│  │ R$ 2,99        │     │ R$ 24,90+      │     │ R$ 19,90/mes   │      │
│  └────────────────┘     └────────────────┘     └────────────────┘      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.2 UI de Conversao

```
┌─────────────────────────────────────────┐
│                                         │
│  🔍 Buscando: "Medicamento XYZ"         │
│                                         │
│  ┌─────────────────────────────────────┐│
│  │                                     ││
│  │  Este medicamento nao esta         ││
│  │  disponivel offline.               ││
│  │                                     ││
│  │  Deseja consultar online?          ││
│  │                                     ││
│  │  ┌─────────────────────────────┐   ││
│  │  │ 💳 Consulta avulsa          │   ││
│  │  │    R$ 2,99                  │   ││
│  │  └─────────────────────────────┘   ││
│  │                                     ││
│  │  ┌─────────────────────────────┐   ││
│  │  │ 📦 Pacote 10 consultas      │   ││
│  │  │    R$ 24,90 (economia 17%)  │   ││
│  │  └─────────────────────────────┘   ││
│  │                                     ││
│  │  [Cancelar]                        ││
│  │                                     ││
│  └─────────────────────────────────────┘│
│                                         │
└─────────────────────────────────────────┘
```

---

## 5. Processamento de Pagamentos

### 5.1 Gateways Suportados

| Gateway | Metodos | Taxa | Liquidacao |
|---------|---------|------|------------|
| Stripe | Cartao, Apple Pay | 2.9% + R$ 0,30 | D+2 |
| Mercado Pago | PIX, Cartao | 0.99% (PIX) / 4.99% (Cartao) | D+0 (PIX) |

### 5.2 Fluxo de Pagamento

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      FLUXO DE PAGAMENTO                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  USUARIO                         BACKEND                 GATEWAY        │
│     │                               │                       │            │
│     │  1. Solicita consulta         │                       │            │
│     │─────────────────────────────▶│                       │            │
│     │                               │                       │            │
│     │  2. Retorna opcoes pagamento  │                       │            │
│     │◀─────────────────────────────│                       │            │
│     │                               │                       │            │
│     │  3. Seleciona metodo (PIX)    │                       │            │
│     │─────────────────────────────▶│                       │            │
│     │                               │  4. Cria pagamento    │            │
│     │                               │─────────────────────▶│            │
│     │                               │                       │            │
│     │                               │  5. QR Code PIX       │            │
│     │                               │◀─────────────────────│            │
│     │  6. Exibe QR Code             │                       │            │
│     │◀─────────────────────────────│                       │            │
│     │                               │                       │            │
│     │  [Usuario paga via app banco] │                       │            │
│     │                               │                       │            │
│     │                               │  7. Webhook: Aprovado │            │
│     │                               │◀─────────────────────│            │
│     │                               │                       │            │
│     │  8. Notifica + libera conteudo│                       │            │
│     │◀─────────────────────────────│                       │            │
│     │                               │                       │            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Implementacao NestJS

```typescript
// payment.service.ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly stripeService: StripeService,
    private readonly paymentRepository: PaymentRepository,
  ) {}

  async createPayment(
    userId: string,
    type: PaymentType,
    method: PaymentMethod,
  ): Promise<PaymentResponse> {
    const amount = this.getAmountByType(type);

    // Criar registro no banco
    const payment = await this.paymentRepository.create({
      userId,
      amount,
      currency: 'BRL',
      status: PaymentStatus.PENDING,
      paymentMethod: method,
      type,
    });

    // Criar no gateway
    if (method === PaymentMethod.PIX) {
      const pixData = await this.stripeService.createPixPayment(amount, payment.id);
      return {
        paymentId: payment.id,
        status: 'PENDING',
        pix: pixData,
      };
    }

    // Cartao: retornar client_secret para Stripe Elements
    const intent = await this.stripeService.createPaymentIntent(amount, payment.id);
    return {
      paymentId: payment.id,
      status: 'PENDING',
      clientSecret: intent.client_secret,
    };
  }

  private getAmountByType(type: PaymentType): number {
    const prices = {
      [PaymentType.SINGLE]: 299,      // R$ 2,99 em centavos
      [PaymentType.PACK_10]: 2490,    // R$ 24,90
      [PaymentType.PACK_25]: 5490,    // R$ 54,90
      [PaymentType.PACK_50]: 9990,    // R$ 99,90
    };
    return prices[type];
  }

  // Webhook handler
  async handleWebhook(event: Stripe.Event): Promise<void> {
    if (event.type === 'payment_intent.succeeded') {
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      const paymentId = paymentIntent.metadata.paymentId;

      await this.paymentRepository.update(paymentId, {
        status: PaymentStatus.APPROVED,
        externalId: paymentIntent.id,
      });

      // Liberar creditos/consulta
      await this.creditService.addCredits(paymentId);
    }
  }
}
```

---

## 6. Sistema de Creditos

### 6.1 Funcionamento

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     SISTEMA DE CREDITOS                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  AQUISICAO                                                               │
│  ┌─────────────────┐                                                    │
│  │ Consulta avulsa │────▶ +1 credito (consumido imediatamente)         │
│  │ Pacote 10       │────▶ +10 creditos                                 │
│  │ Pacote 25       │────▶ +25 creditos                                 │
│  │ Pacote 50       │────▶ +50 creditos                                 │
│  └─────────────────┘                                                    │
│                                                                          │
│  CONSUMO                                                                 │
│  ┌─────────────────┐                                                    │
│  │ Consulta online │────▶ -1 credito                                   │
│  └─────────────────┘                                                    │
│                                                                          │
│  VALIDADE                                                                │
│  ┌─────────────────┐                                                    │
│  │ Pacotes         │────▶ 12 meses a partir da compra                  │
│  │ Consulta avulsa │────▶ Imediato (sem estoque)                       │
│  └─────────────────┘                                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Modelo de Dados

```typescript
// credit.entity.ts
@Entity('med_user_credit')
export class UserCredit {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ name: 'user_id' })
  userId: string;

  @Column({ type: 'int' })
  balance: number;

  @Column({ name: 'expires_at', type: 'timestamp' })
  expiresAt: Date;

  @Column({ name: 'payment_id' })
  paymentId: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;
}

// credit.service.ts
@Injectable()
export class CreditService {
  async hasCredits(userId: string): Promise<boolean> {
    const totalCredits = await this.creditRepository
      .createQueryBuilder('credit')
      .where('credit.userId = :userId', { userId })
      .andWhere('credit.balance > 0')
      .andWhere('credit.expiresAt > :now', { now: new Date() })
      .select('SUM(credit.balance)', 'total')
      .getRawOne();

    return totalCredits?.total > 0;
  }

  async consumeCredit(userId: string): Promise<void> {
    // Busca credito mais antigo (FIFO)
    const credit = await this.creditRepository.findOne({
      where: {
        userId,
        balance: MoreThan(0),
        expiresAt: MoreThan(new Date()),
      },
      order: { createdAt: 'ASC' },
    });

    if (!credit) {
      throw new InsufficientCreditsException();
    }

    credit.balance -= 1;
    await this.creditRepository.save(credit);
  }
}
```

---

## 7. Metricas de Monetizacao

### 7.1 KPIs Principais

| Metrica | Formula | Meta |
|---------|---------|------|
| ARPU | Receita Total / Usuarios Ativos | R$ 2,50/mes |
| Conversion Rate | Pagantes / Total Usuarios | 3-5% |
| LTV | ARPU x Tempo Medio Retencao | R$ 30 |
| CAC | Custo Marketing / Novos Usuarios | < R$ 10 |
| LTV:CAC Ratio | LTV / CAC | > 3:1 |

### 7.2 Funil de Conversao

```
Download          100.000  (100%)
     │
     ▼
Cadastro           60.000  (60%)
     │
     ▼
Busca offline      45.000  (45%)
     │
     ▼
Busca NOT offline  15.000  (15%)  ◀── Gatilho de conversao
     │
     ▼
Inicia pagamento    3.000  (3%)
     │
     ▼
Pagamento OK        2.400  (2.4%)  ◀── Taxa de conversao
```

---

## 8. Roadmap de Monetizacao

### Fase 1 - MVP (Atual)
- [x] Modelo freemium basico
- [x] Consultas avulsas (pay-per-use)
- [x] Pacotes de consultas
- [ ] Integracao PIX
- [ ] Integracao Cartao

### Fase 2 - Expansao (6 meses)
- [ ] Assinatura premium mensal/anual
- [ ] Alertas de interacao (premium)
- [ ] Exportacao de dados (premium)
- [ ] Promocoes sazonais

### Fase 3 - B2B (12 meses)
- [ ] Licenciamento para clinicas
- [ ] API para integracao com sistemas
- [ ] Planos corporativos
- [ ] White-label para hospitais

---

## 9. Aspectos Legais

### 9.1 Nota Fiscal

- Emissao automatica via API (Nota Carioca, NFe.io, etc.)
- CNAE: 6311-9/00 (Tratamento de dados)
- Regime: Simples Nacional ou Lucro Presumido

### 9.2 Politica de Reembolso

| Situacao | Reembolso |
|----------|-----------|
| Erro tecnico | 100% |
| Desistencia antes do uso | 100% |
| Creditos nao utilizados (< 7 dias) | Proporcional |
| Creditos expirados | Nao |

### 9.3 Termos de Pagamento

- Todos os precos em BRL
- Impostos inclusos
- Sem cobranca recorrente automatica (pacotes sao one-time)
- Assinaturas com renovacao automatica (com aviso 7 dias antes)

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
