# Requisitos - MedInfo

## 1. Requisitos Funcionais

### RF01 - Bula Digital Adaptativa

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF01.1 | O sistema deve manter base de dados local com no minimo 500 medicamentos comuns | Alta | Pendente |
| RF01.2 | O sistema deve exibir no minimo 3 marcas comerciais por medicamento | Alta | Pendente |
| RF01.3 | O sistema deve permitir consulta offline para medicamentos no banco local | Alta | Pendente |
| RF01.4 | O sistema deve permitir consulta online (paga) para medicamentos nao disponiveis localmente | Media | Pendente |
| RF01.5 | O sistema deve apresentar **Visao Tecnica** para profissionais e estudantes | Alta | Pendente |
| RF01.6 | O sistema deve apresentar **Visao Simplificada** para publico geral | Alta | Pendente |
| RF01.7 | O sistema deve permitir troca entre visoes para usuarios validados | Media | Pendente |

#### RF01.5.1 - Visao Tecnica (Detalhamento)
- Informacoes farmacologicas completas
- Composicao detalhada (principio ativo, excipientes)
- Farmacocinetica (absorcao, distribuicao, metabolismo, excrecao)
- Farmacodinamica (mecanismo de acao)
- Interacoes medicamentosas
- Contraindicacoes tecnicas
- Posologia detalhada por indicacao

#### RF01.6.1 - Visao Simplificada (Detalhamento)
- Linguagem acessivel e didatica (nivel de leitura 8a serie)
- Informacoes essenciais de uso ("Para que serve")
- Dosagem simplificada ("Como tomar")
- Efeitos colaterais comuns em linguagem clara
- Alertas importantes destacados visualmente
- Icones e cores para facilitar compreensao

### RF02 - Calculadora de Conversao mg/ml

Calculadora simples para converter miligramas (mg) em mililitros (ml) com base na concentracao do medicamento.

**Formula:**
```
Volume (ml) = Dose desejada (mg) / Concentracao (mg/ml)
```

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF02.1 | A calculadora deve ser exclusiva para usuarios Profissional ou Estudante | Alta | Pendente |
| RF02.2 | O sistema deve validar credenciais antes de liberar acesso a calculadora | Alta | Pendente |
| RF02.3 | A calculadora deve funcionar offline apos validacao inicial | Alta | Pendente |
| RF02.4 | A calculadora deve aceitar dose desejada em mg | Alta | Pendente |
| RF02.5 | A calculadora deve aceitar concentracao do medicamento em mg/ml | Alta | Pendente |
| RF02.6 | A calculadora deve calcular e exibir o volume em ml | Alta | Pendente |
| RF02.7 | A calculadora deve exibir a formula utilizada | Baixa | Pendente |

#### RF02.4.1 - Parametros da Calculadora
| Parametro | Tipo | Obrigatorio | Validacao | Exemplo |
|-----------|------|-------------|-----------|---------|
| Dose desejada | Decimal (mg) | Sim | > 0 | 500 mg |
| Concentracao | Decimal (mg/ml) | Sim | > 0 | 50 mg/ml |

#### RF02.6.1 - Saida da Calculadora
| Campo | Tipo | Exemplo |
|-------|------|---------|
| Volume calculado | Decimal (ml) | 10 ml |
| Formula aplicada | String | "500 mg / 50 mg/ml = 10 ml" |

#### Exemplo de Uso
```
Entrada:
  - Dose desejada: 500 mg
  - Concentracao: 50 mg/ml

Calculo:
  Volume = 500 / 50 = 10 ml

Saida:
  - Volume: 10 ml
  - Formula: "500 mg / 50 mg/ml = 10 ml"
```

### RF03 - Sistema de Cadastro e Autenticacao

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF03.1 | O cadastro deve ser online obrigatoriamente | Alta | Pendente |
| RF03.2 | O sistema deve suportar 3 perfis: Profissional, Estudante, Publico Geral | Alta | Pendente |
| RF03.3 | O sistema deve validar credenciais de Profissionais (CRM, CRF, COREN, etc.) | Alta | Pendente |
| RF03.4 | O sistema deve validar matricula academica de Estudantes | Alta | Pendente |
| RF03.5 | O cadastro de Publico Geral deve ser simplificado (email + senha) | Alta | Pendente |
| RF03.6 | O sistema deve permitir login com biometria apos primeiro acesso | Media | Pendente |
| RF03.7 | O sistema deve manter sessao ativa para uso offline | Alta | Pendente |

#### RF03.3.1 - Validacao de Profissionais
| Conselho | Campos | Validacao |
|----------|--------|-----------|
| CRM (Medicos) | UF + Numero | API CRM ou manual |
| CRF (Farmaceuticos) | UF + Numero | API CRF ou manual |
| COREN (Enfermeiros) | UF + Numero | API COREN ou manual |
| CRO (Dentistas) | UF + Numero | API CRO ou manual |
| CRBM (Biomedicos) | UF + Numero | Manual |

### RF04 - Sistema de Busca

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF04.1 | O sistema deve permitir busca por nome comercial | Alta | Pendente |
| RF04.2 | O sistema deve permitir busca por principio ativo | Alta | Pendente |
| RF04.3 | O sistema deve permitir busca por codigo de barras (scan) | Media | Pendente |
| RF04.4 | O sistema deve sugerir medicamentos durante digitacao (autocomplete) | Media | Pendente |
| RF04.5 | O sistema deve exibir historico de buscas recentes | Baixa | Pendente |
| RF04.6 | O sistema deve permitir favoritar medicamentos | Baixa | Pendente |

### RF05 - Sistema de Pagamento

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF05.1 | O sistema deve processar pagamentos para consultas online | Media | Pendente |
| RF05.2 | O sistema deve suportar cartao de credito/debito | Media | Pendente |
| RF05.3 | O sistema deve suportar PIX | Media | Pendente |
| RF05.4 | O sistema deve oferecer pacotes de consultas | Baixa | Pendente |
| RF05.5 | O sistema deve emitir recibo de pagamento | Media | Pendente |

---

## 2. Requisitos Nao-Funcionais

### RNF01 - Performance

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF01.1 | Tempo de busca offline | Latencia | < 200ms |
| RNF01.2 | Tempo de carregamento de bula | Latencia | < 500ms |
| RNF01.3 | Tempo de calculo de dosagem | Latencia | < 100ms |
| RNF01.4 | Tamanho do banco local | Armazenamento | < 100MB |
| RNF01.5 | Consumo de bateria em uso | Energia | < 5%/hora |

### RNF02 - Disponibilidade e Confiabilidade

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF02.1 | Disponibilidade do backend | Uptime | 99.5% |
| RNF02.2 | Funcionamento offline | Disponibilidade | 100% para base local |
| RNF02.3 | Sincronizacao de dados | Consistencia | Eventual (24h max) |
| RNF02.4 | Backup de dados do usuario | RPO | 24 horas |

### RNF03 - Seguranca

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF03.1 | Criptografia de dados sensiveis | Algoritmo | AES-256 |
| RNF03.2 | Comunicacao com backend | Protocolo | TLS 1.3 |
| RNF03.3 | Armazenamento de credenciais | Metodo | Keystore Android |
| RNF03.4 | Tokens de autenticacao | Expiracao | 7 dias (refresh: 30 dias) |
| RNF03.5 | Conformidade LGPD | Compliance | 100% |

### RNF04 - Usabilidade

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF04.1 | Acessibilidade | WCAG | Nivel AA |
| RNF04.2 | Suporte a tamanhos de fonte | Range | 80% - 200% |
| RNF04.3 | Suporte a modo escuro | Temas | Claro/Escuro/Sistema |
| RNF04.4 | Idioma | Localizacao | pt-BR (inicial) |

### RNF05 - Compatibilidade

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF05.1 | Versao minima Android | API Level | 26 (Android 8.0) |
| RNF05.2 | Versao alvo Android | API Level | 34 (Android 14) |
| RNF05.3 | Resolucoes suportadas | Range | 320dp - 600dp width |
| RNF05.4 | Orientacao | Suporte | Portrait (inicial) |

### RNF06 - Escalabilidade

| ID | Requisito | Metrica | Meta |
|----|-----------|---------|------|
| RNF06.1 | Usuarios simultaneos | Capacidade | 10.000 |
| RNF06.2 | Requests por segundo | Throughput | 1.000 RPS |
| RNF06.3 | Crescimento de base | Escalabilidade | Horizontal |

---

## 3. Regras de Negocio

### RN01 - Perfis de Usuario

| Regra | Descricao |
|-------|-----------|
| RN01.1 | Perfil e definido no cadastro e pode ser alterado mediante nova validacao |
| RN01.2 | Profissional deve ter registro ativo no conselho de classe |
| RN01.3 | Estudante deve ter matricula ativa em instituicao reconhecida pelo MEC |
| RN01.4 | Publico Geral nao tem acesso a calculadora de dosagem |

### RN02 - Acesso a Funcionalidades

| Funcionalidade | Publico Geral | Estudante | Profissional |
|----------------|---------------|-----------|--------------|
| Bula Simplificada | Sim | Sim | Sim |
| Bula Tecnica | Nao | Sim | Sim |
| Calculadora Dosagem | Nao | Sim | Sim |
| Consulta Offline | Sim | Sim | Sim |
| Consulta Online (paga) | Sim | Sim | Sim |

### RN03 - Monetizacao

| Regra | Descricao |
|-------|-----------|
| RN03.1 | Consultas a medicamentos no banco local sao gratuitas |
| RN03.2 | Consultas online a medicamentos externos sao cobradas por consulta |
| RN03.3 | Preco por consulta: R$ X,XX (definir) |
| RN03.4 | Pacotes de consultas tem desconto progressivo |

### RN04 - Dados e Atualizacoes

| Regra | Descricao |
|-------|-----------|
| RN04.1 | Base local e atualizada mensalmente |
| RN04.2 | Usuario deve ter conexao para receber atualizacoes |
| RN04.3 | Atualizacoes sao incrementais (delta) para economia de dados |
| RN04.4 | Fonte oficial de dados: ANVISA |

---

## 4. Casos de Uso Principais

### UC01 - Consultar Bula de Medicamento

```
Ator: Usuario (qualquer perfil)
Pre-condicao: Usuario autenticado
Fluxo Principal:
1. Usuario acessa tela de busca
2. Usuario digita nome do medicamento
3. Sistema exibe sugestoes (autocomplete)
4. Usuario seleciona medicamento
5. Sistema identifica perfil do usuario
6. Sistema exibe bula na visao apropriada

Fluxo Alternativo A (Medicamento nao encontrado localmente):
4a. Sistema informa que medicamento nao esta na base local
5a. Sistema oferece consulta online (paga)
6a. Usuario confirma pagamento
7a. Sistema consulta base externa e exibe resultado

Fluxo Alternativo B (Usuario offline e medicamento nao local):
4b. Sistema informa que medicamento requer conexao
5b. Sistema sugere medicamentos similares disponiveis offline
```

### UC02 - Calcular Dosagem

```
Ator: Profissional ou Estudante (validado)
Pre-condicao: Usuario validado, medicamento selecionado
Fluxo Principal:
1. Usuario acessa calculadora de dosagem
2. Usuario informa parametros do paciente (peso, altura, sexo, idade)
3. Usuario seleciona medicamento
4. Sistema exibe campos adicionais se necessario
5. Usuario confirma calculo
6. Sistema calcula e exibe dose recomendada
7. Sistema exibe formula utilizada

Fluxo Alternativo A (Parametros invalidos):
5a. Sistema valida parametros
6a. Sistema exibe erros de validacao
7a. Usuario corrige e resubmete
```

### UC03 - Validar Credenciais de Profissional

```
Ator: Usuario (cadastro como Profissional)
Pre-condicao: Conexao com internet
Fluxo Principal:
1. Usuario seleciona perfil "Profissional"
2. Sistema solicita tipo de conselho (CRM, CRF, COREN, etc.)
3. Usuario informa UF e numero de registro
4. Sistema consulta API do conselho (se disponivel)
5. Sistema valida e confirma registro ativo
6. Sistema libera acesso a funcionalidades de profissional

Fluxo Alternativo A (API indisponivel):
4a. Sistema informa que validacao automatica indisponivel
5a. Sistema solicita upload de documento comprobatorio
6a. Usuario envia foto do registro
7a. Sistema envia para validacao manual
8a. Sistema notifica usuario quando aprovado (ate 48h)

Fluxo Alternativo B (Registro invalido/inativo):
5b. Sistema informa que registro nao foi encontrado ou esta inativo
6b. Usuario pode tentar novamente ou cadastrar como Publico Geral
```

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
