# Arquitetura - MedInfo

## 1. Visao Geral da Arquitetura

O MedInfo e composto por duas aplicacoes principais que se comunicam via API REST:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              MOBILE (Android)                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐  │
│  │ Presentation│───▶│   Domain    │◀───│           Data              │  │
│  │   (MVVM)    │    │  (UseCases) │    │ (Repository Implementations)│  │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘  │
│                                               │          │               │
│                                        ┌──────┴──┐  ┌────┴────┐         │
│                                        │  Room   │  │ Retrofit│         │
│                                        │ (Local) │  │ (Remote)│         │
│                                        └─────────┘  └────┬────┘         │
└────────────────────────────────────────────────────────────│─────────────┘
                                                             │
                                                             │ HTTPS
                                                             ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           BACKEND (NestJS)                               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐  │
│  │ Controllers │───▶│  Services   │───▶│        Repositories         │  │
│  │   (REST)    │    │ (Business)  │    │      (TypeORM/Prisma)       │  │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘  │
│                                                       │                  │
│                                               ┌───────┴───────┐         │
│                                               │  PostgreSQL   │         │
│                                               │    + Redis    │         │
│                                               └───────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 2. Arquitetura Mobile (MVVM + Clean Architecture)

### 2.1 Camadas

```
┌──────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │  Composables │  │  ViewModels  │  │  UI State/Events │    │
│  │   (Screens)  │  │   (MVVM)     │  │                  │    │
│  └──────────────┘  └──────────────┘  └──────────────────┘    │
│                           │                                   │
│                           ▼                                   │
├──────────────────────────────────────────────────────────────┤
│                      DOMAIN LAYER                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │   UseCases   │  │    Models    │  │ Repository Iface │    │
│  │              │  │   (Domain)   │  │                  │    │
│  └──────────────┘  └──────────────┘  └──────────────────┘    │
│                           │                                   │
│                           ▼                                   │
├──────────────────────────────────────────────────────────────┤
│                       DATA LAYER                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                  Repository Impl                        │  │
│  │  ┌─────────────────────┐  ┌─────────────────────────┐  │  │
│  │  │    Local Source     │  │     Remote Source       │  │  │
│  │  │  ┌──────┐ ┌──────┐  │  │  ┌───────┐ ┌────────┐  │  │  │
│  │  │  │ Room │ │DataSt│  │  │  │Retrofit│ │  DTOs  │  │  │  │
│  │  │  └──────┘ └──────┘  │  │  └───────┘ └────────┘  │  │  │
│  │  └─────────────────────┘  └─────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Fluxo de Dados (Offline-First)

```
Buscar Medicamento:

1. ViewModel chama UseCase
2. UseCase chama Repository (interface)
3. Repository (impl) decide fonte:

   ┌─────────────────────────────────────────────────────────┐
   │                     Repository                           │
   │                                                          │
   │   searchMedication(query):                               │
   │     1. Busca no Room (local)                            │
   │        ├─ Encontrado? ─────▶ Retorna resultado local    │
   │        │                                                 │
   │        └─ Nao encontrado?                               │
   │           │                                              │
   │           ├─ Sem conexao? ──▶ Retorna "nao disponivel"  │
   │           │                                              │
   │           └─ Com conexao? ──▶ Busca na API remota       │
   │                              │                           │
   │                              └─▶ Retorna resultado       │
   │                                  (usuario paga se nao   │
   │                                   estiver no plano)     │
   └─────────────────────────────────────────────────────────┘
```

### 2.3 Gerenciamento de Estado

```kotlin
// UI State (imutavel)
sealed class MedicationSearchState {
    object Idle : MedicationSearchState()
    object Loading : MedicationSearchState()
    data class Success(
        val medications: List<MedicationSummary>,
        val source: DataSource // LOCAL ou REMOTE
    ) : MedicationSearchState()
    data class Error(
        val type: ErrorType,
        val message: String
    ) : MedicationSearchState()
}

enum class DataSource { LOCAL, REMOTE }
enum class ErrorType { NETWORK, NOT_FOUND, PAYMENT_REQUIRED, UNKNOWN }

// UI Events (acoes do usuario)
sealed class MedicationSearchEvent {
    data class Search(val query: String) : MedicationSearchEvent()
    data class SelectMedication(val id: String) : MedicationSearchEvent()
    object ClearSearch : MedicationSearchEvent()
    object Retry : MedicationSearchEvent()
}
```

## 3. Arquitetura Backend (NestJS + Clean Architecture)

### 3.1 Estrutura de Modulos

```
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Guards │ Interceptors │ Filters │ Pipes             │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      MODULES                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │    Auth     │ │ Medication  │ │    User/Profile     │   │
│  │   Module    │ │   Module    │ │      Module         │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Dosage    │ │  Payment    │ │   Notification      │   │
│  │   Module    │ │   Module    │ │      Module         │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      SHARED                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Config    │ │  Database   │ │      Common         │   │
│  │   Module    │ │   Module    │ │      Module         │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Estrutura de um Modulo

```
medication/
├── dto/
│   ├── create-medication.dto.ts
│   ├── update-medication.dto.ts
│   ├── medication-response.dto.ts
│   └── search-medication.dto.ts
├── entities/
│   ├── medication.entity.ts
│   └── brand.entity.ts
├── interfaces/
│   └── medication-repository.interface.ts
├── repositories/
│   └── medication.repository.ts
├── medication.controller.ts
├── medication.service.ts
├── medication.module.ts
└── medication.constants.ts
```

### 3.3 Fluxo de Request

```
HTTP Request
    │
    ▼
┌──────────────────┐
│    Middleware    │──▶ Logging, CORS, Rate Limit
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      Guard       │──▶ Authentication (JWT)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Interceptor    │──▶ Transform Response, Cache
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      Pipe        │──▶ Validation (class-validator)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Controller     │──▶ Route Handler
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Service       │──▶ Business Logic
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Repository     │──▶ Data Access
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Database      │
└──────────────────┘
```

## 4. Comunicacao Mobile-Backend

### 4.1 Autenticacao

```
┌────────────────┐                    ┌────────────────┐
│     Mobile     │                    │    Backend     │
└───────┬────────┘                    └───────┬────────┘
        │                                     │
        │  POST /auth/login                   │
        │  {email, password}                  │
        │────────────────────────────────────▶│
        │                                     │
        │  200 OK                             │
        │  {accessToken, refreshToken}        │
        │◀────────────────────────────────────│
        │                                     │
        │  GET /medications (with Bearer)     │
        │  Authorization: Bearer {token}      │
        │────────────────────────────────────▶│
        │                                     │
        │  200 OK                             │
        │  {data: [...]}                      │
        │◀────────────────────────────────────│
        │                                     │
        │  Token Expired?                     │
        │  POST /auth/refresh                 │
        │  {refreshToken}                     │
        │────────────────────────────────────▶│
        │                                     │
        │  200 OK                             │
        │  {accessToken, refreshToken}        │
        │◀────────────────────────────────────│
```

### 4.2 Sincronizacao Offline

```
┌─────────────────────────────────────────────────────────────┐
│                   SYNC STRATEGY                              │
│                                                              │
│  1. INITIAL SYNC (primeiro acesso com conexao):             │
│     - Download base completa de medicamentos locais         │
│     - ~50-100MB comprimido                                  │
│     - Armazenado no Room                                    │
│                                                              │
│  2. INCREMENTAL SYNC (atualizacoes posteriores):            │
│     - Verifica lastSyncTimestamp                            │
│     - Download apenas deltas (novos/alterados/removidos)    │
│     - ~1-5MB por atualizacao mensal                         │
│                                                              │
│  3. CONFLICT RESOLUTION:                                     │
│     - Server wins (dados de medicamentos sao read-only)     │
│     - Favoritos do usuario: merge com timestamp             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 5. Infraestrutura

### 5.1 Diagrama de Deployment

```
┌───────────────────────────────────────────────────────────────────┐
│                           CLOUD                                    │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                      Load Balancer                            │ │
│  │                    (AWS ALB / nginx)                          │ │
│  └─────────────────────────┬────────────────────────────────────┘ │
│                            │                                       │
│         ┌──────────────────┼──────────────────┐                   │
│         │                  │                  │                    │
│         ▼                  ▼                  ▼                    │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐              │
│  │  API Pod 1 │    │  API Pod 2 │    │  API Pod N │              │
│  │  (NestJS)  │    │  (NestJS)  │    │  (NestJS)  │              │
│  └─────┬──────┘    └─────┬──────┘    └─────┬──────┘              │
│        │                 │                 │                       │
│        └─────────────────┼─────────────────┘                      │
│                          │                                         │
│         ┌────────────────┼────────────────┐                       │
│         │                │                │                        │
│         ▼                ▼                ▼                        │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐                │
│  │ PostgreSQL │   │   Redis    │   │    S3      │                │
│  │  (RDS)     │   │  (Cache)   │   │  (Storage) │                │
│  └────────────┘   └────────────┘   └────────────┘                │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────┐
│                          MOBILE                                    │
│                                                                    │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐                │
│  │   User 1   │   │   User 2   │   │   User N   │                │
│  │  (Android) │   │  (Android) │   │  (Android) │                │
│  └────────────┘   └────────────┘   └────────────┘                │
│        │                │                │                         │
│        │   ┌────────────┼────────────┐   │                        │
│        │   │            │            │   │                         │
│        │   ▼            ▼            ▼   │                         │
│        │  ┌─────────────────────────────┐│                        │
│        │  │      Local Database         ││                        │
│        │  │         (Room)              ││                        │
│        │  └─────────────────────────────┘│                        │
│        │                                  │                        │
│        └────── HTTPS / REST API ─────────┘                        │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘
```

### 5.2 Stack de Infraestrutura

| Componente | Tecnologia | Observacoes |
|------------|------------|-------------|
| Cloud Provider | AWS / GCP / Azure | Escolher conforme custo/expertise |
| Container Orchestration | Kubernetes (EKS) | ou ECS para simplificar |
| Database | PostgreSQL (RDS) | Managed service |
| Cache | Redis (ElastiCache) | Session, cache de queries |
| Storage | S3 | Imagens, arquivos estaticos |
| CDN | CloudFront | Assets do app |
| CI/CD | GitHub Actions | ou GitLab CI |
| Monitoring | Datadog / NewRelic | APM + Logs + Metrics |
| APM Mobile | Firebase Crashlytics | Crashes e ANRs |

## 6. Decisoes Arquiteturais (ADRs)

### ADR-001: MVVM para Mobile

**Contexto**: Escolher padrao arquitetural para o app Android.

**Decisao**: MVVM com Jetpack Compose.

**Justificativa**:
- Padrao recomendado pelo Google para Android
- Excelente integracao com Compose e StateFlow
- Separacao clara entre UI e logica
- Facilita testes unitarios

**Consequencias**:
- (+) Codigo testavel e manutenivel
- (+) Reatividade nativa com Flow
- (-) Curva de aprendizado para Compose

### ADR-002: Offline-First

**Contexto**: App precisa funcionar sem conexao.

**Decisao**: Arquitetura Offline-First com Room como fonte primaria.

**Justificativa**:
- Requisito central do produto
- Usuarios em areas com conectividade limitada
- Melhor UX (respostas instantaneas)

**Consequencias**:
- (+) Funcionamento garantido offline
- (+) Performance superior
- (-) Complexidade de sincronizacao
- (-) Maior uso de armazenamento

### ADR-003: NestJS para Backend

**Contexto**: Escolher framework para API REST.

**Decisao**: NestJS com TypeScript.

**Justificativa**:
- Arquitetura modular e opinativa
- TypeScript para type-safety
- Excelente ecossistema (TypeORM, Passport, Swagger)
- Boa documentacao e comunidade

**Consequencias**:
- (+) Codigo organizado e padronizado
- (+) Geracao automatica de docs (Swagger)
- (-) Overhead vs Express puro
- (-) Equipe precisa conhecer decorators/DI

### ADR-004: PostgreSQL como Database

**Contexto**: Escolher banco de dados principal.

**Decisao**: PostgreSQL 16.

**Justificativa**:
- Robusto e confiavel
- Excelente suporte a JSON (dados farmacologicos)
- Full-text search nativo
- Open source, sem vendor lock-in

**Consequencias**:
- (+) Queries complexas eficientes
- (+) JSONB para dados semi-estruturados
- (-) Mais complexo que NoSQL para escalar horizontalmente

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
