# Modelo de Dados - MedInfo

## 1. Visao Geral

O MedInfo utiliza PostgreSQL 16 como banco de dados principal, com schema `medinfo` para organizar todas as tabelas.

### Nomenclatura

| Elemento | Convencao | Exemplo |
|----------|-----------|---------|
| Schema | `medinfo` | `medinfo.med_user` |
| Tabelas | `med_{nome}` (singular) | `med_medication` |
| Colunas | `snake_case` | `commercial_name` |
| PKs | `id` (UUID) | `id` |
| FKs | `{tabela}_id` | `medication_id` |

## 2. Diagrama ER

```
┌─────────────────────┐       ┌─────────────────────┐
│     med_user        │       │  med_professional   │
├─────────────────────┤       │     _validation     │
│ id (PK)             │──────▶├─────────────────────┤
│ email               │       │ id (PK)             │
│ password_hash       │       │ user_id (FK)        │
│ name                │       │ council_type        │
│ role                │       │ council_number      │
│ is_validated        │       │ council_state       │
│ created_at          │       │ status              │
│ updated_at          │       │ validated_at        │
└─────────────────────┘       │ document_url        │
         │                    └─────────────────────┘
         │
         ▼
┌─────────────────────┐       ┌─────────────────────┐
│   med_favorite      │       │    med_payment      │
├─────────────────────┤       ├─────────────────────┤
│ id (PK)             │       │ id (PK)             │
│ user_id (FK)        │       │ user_id (FK)        │
│ medication_id (FK)  │──────▶│ amount              │
│ created_at          │       │ currency            │
└─────────────────────┘       │ status              │
         │                    │ payment_method      │
         │                    │ created_at          │
         ▼                    └─────────────────────┘
┌─────────────────────┐                │
│   med_medication    │                │
├─────────────────────┤                ▼
│ id (PK)             │◀───────┌─────────────────────┐
│ commercial_name     │        │ med_online_query    │
│ active_ingredient   │        ├─────────────────────┤
│ description         │        │ id (PK)             │
│ is_offline          │        │ user_id (FK)        │
│ technical_info      │        │ medication_id (FK)  │
│ simplified_info     │        │ payment_id (FK)     │
│ created_at          │        │ query_type          │
│ updated_at          │        │ created_at          │
└─────────────────────┘        └─────────────────────┘
         │
         │
         ▼
┌─────────────────────┐       ┌─────────────────────┐
│     med_brand       │       │ med_interaction     │
├─────────────────────┤       ├─────────────────────┤
│ id (PK)             │       │ id (PK)             │
│ medication_id (FK)  │       │ medication_a_id(FK) │
│ name                │       │ medication_b_id(FK) │
│ manufacturer        │       │ severity            │
│ barcode             │       │ description         │
│ created_at          │       └─────────────────────┘
└─────────────────────┘

┌─────────────────────┐
│  med_search_log     │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ query               │
│ results_count       │
│ source              │
│ created_at          │
└─────────────────────┘

Nota: A calculadora de conversao mg/ml nao requer tabela
no banco de dados - e um calculo simples feito no cliente.
```

## 3. Definicao das Tabelas

### 3.1 med_user

Armazena informacoes dos usuarios do sistema.

```sql
CREATE TABLE medinfo.med_user (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(150) NOT NULL,
    role VARCHAR(20) NOT NULL,
    is_validated BOOLEAN DEFAULT FALSE,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_user PRIMARY KEY (id),
    CONSTRAINT uq_user_email UNIQUE (email),
    CONSTRAINT ck_user_role CHECK (role IN ('PROFESSIONAL', 'STUDENT', 'GENERAL'))
);

CREATE INDEX idx_user_email ON medinfo.med_user(email);
CREATE INDEX idx_user_role ON medinfo.med_user(role);
```

| Coluna | Tipo | Obrigatorio | Descricao |
|--------|------|-------------|-----------|
| id | UUID | Sim | Identificador unico |
| email | VARCHAR(255) | Sim | Email do usuario (unico) |
| password_hash | VARCHAR(255) | Sim | Hash da senha (bcrypt) |
| name | VARCHAR(150) | Sim | Nome completo |
| role | VARCHAR(20) | Sim | Perfil: PROFESSIONAL, STUDENT, GENERAL |
| is_validated | BOOLEAN | Nao | Se credenciais foram validadas |
| last_login_at | TIMESTAMP | Nao | Ultimo acesso |
| created_at | TIMESTAMP | Sim | Data de criacao |
| updated_at | TIMESTAMP | Sim | Data de atualizacao |

### 3.2 med_professional_validation

Armazena dados de validacao de profissionais e estudantes.

```sql
CREATE TABLE medinfo.med_professional_validation (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    council_type VARCHAR(10) NOT NULL,
    council_number VARCHAR(20) NOT NULL,
    council_state CHAR(2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING',
    validated_at TIMESTAMP,
    rejected_reason TEXT,
    document_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_professional_validation PRIMARY KEY (id),
    CONSTRAINT fk_validation_user FOREIGN KEY (user_id)
        REFERENCES medinfo.med_user(id) ON DELETE CASCADE,
    CONSTRAINT ck_validation_council_type CHECK (
        council_type IN ('CRM', 'CRF', 'COREN', 'CRO', 'CRBM', 'STUDENT')
    ),
    CONSTRAINT ck_validation_status CHECK (
        status IN ('PENDING', 'APPROVED', 'REJECTED', 'EXPIRED')
    )
);

CREATE INDEX idx_validation_user_id ON medinfo.med_professional_validation(user_id);
CREATE INDEX idx_validation_status ON medinfo.med_professional_validation(status);
```

| Coluna | Tipo | Obrigatorio | Descricao |
|--------|------|-------------|-----------|
| id | UUID | Sim | Identificador unico |
| user_id | UUID | Sim | FK para med_user |
| council_type | VARCHAR(10) | Sim | Tipo: CRM, CRF, COREN, CRO, CRBM, STUDENT |
| council_number | VARCHAR(20) | Sim | Numero do registro/matricula |
| council_state | CHAR(2) | Sim | UF do conselho |
| status | VARCHAR(20) | Sim | PENDING, APPROVED, REJECTED, EXPIRED |
| validated_at | TIMESTAMP | Nao | Data da validacao |
| rejected_reason | TEXT | Nao | Motivo da rejeicao |
| document_url | VARCHAR(500) | Nao | URL do documento enviado |

### 3.3 med_medication

Armazena informacoes dos medicamentos.

```sql
CREATE TABLE medinfo.med_medication (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    commercial_name VARCHAR(255) NOT NULL,
    active_ingredient VARCHAR(255) NOT NULL,
    description TEXT,
    is_offline BOOLEAN DEFAULT TRUE,
    technical_info JSONB,
    simplified_info JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_medication PRIMARY KEY (id)
);

CREATE INDEX idx_medication_commercial_name ON medinfo.med_medication(commercial_name);
CREATE INDEX idx_medication_active_ingredient ON medinfo.med_medication(active_ingredient);
CREATE INDEX idx_medication_is_offline ON medinfo.med_medication(is_offline);

-- Full-text search
CREATE INDEX idx_medication_search ON medinfo.med_medication
    USING GIN (to_tsvector('portuguese', commercial_name || ' ' || active_ingredient));
```

| Coluna | Tipo | Obrigatorio | Descricao |
|--------|------|-------------|-----------|
| id | UUID | Sim | Identificador unico |
| commercial_name | VARCHAR(255) | Sim | Nome comercial principal |
| active_ingredient | VARCHAR(255) | Sim | Principio ativo |
| description | TEXT | Nao | Descricao geral |
| is_offline | BOOLEAN | Sim | Disponivel no banco local |
| technical_info | JSONB | Nao | Informacoes tecnicas (bula completa) |
| simplified_info | JSONB | Nao | Informacoes simplificadas |

#### Estrutura do JSONB technical_info

```json
{
  "composition": {
    "activeIngredients": [
      {"name": "Paracetamol", "amount": "500", "unit": "mg"}
    ],
    "excipients": ["Amido", "Celulose microcristalina"]
  },
  "pharmacokinetics": {
    "absorption": "Rapida absorcao pelo trato gastrointestinal...",
    "distribution": "Distribui-se amplamente pelos tecidos...",
    "metabolism": "Metabolizado principalmente no figado...",
    "excretion": "Excretado principalmente pelos rins..."
  },
  "pharmacodynamics": {
    "mechanismOfAction": "Inibe a sintese de prostaglandinas...",
    "therapeuticClass": "Analgesico e antipiretico"
  },
  "indications": ["Dor leve a moderada", "Febre"],
  "contraindications": ["Hipersensibilidade ao paracetamol", "Insuficiencia hepatica grave"],
  "dosage": {
    "adults": "500mg a 1000mg a cada 4-6 horas",
    "children": "10-15mg/kg a cada 4-6 horas",
    "maxDaily": "4000mg"
  },
  "interactions": [
    {"drug": "Varfarina", "effect": "Aumento do efeito anticoagulante", "severity": "MODERATE"}
  ],
  "adverseReactions": {
    "common": ["Nausea", "Erupcao cutanea"],
    "uncommon": ["Hepatotoxicidade em doses elevadas"],
    "rare": ["Reacoes anafilaticas"]
  }
}
```

#### Estrutura do JSONB simplified_info

```json
{
  "whatIsIt": "Medicamento para aliviar dor e febre",
  "whatIsItFor": [
    "Dor de cabeca",
    "Dor no corpo",
    "Febre"
  ],
  "howToUse": {
    "instructions": "Tomar com agua, de preferencia apos as refeicoes",
    "adults": "1 a 2 comprimidos a cada 4 ou 6 horas",
    "maxPerDay": "Nao ultrapassar 8 comprimidos em 24 horas"
  },
  "warnings": [
    "Nao use se tiver alergia ao paracetamol",
    "Nao use com bebidas alcoolicas",
    "Consulte um medico se a dor persistir por mais de 3 dias"
  ],
  "sideEffects": [
    "Raramente pode causar enjoo",
    "Em casos raros, reacoes na pele"
  ],
  "importantAlerts": [
    {
      "icon": "warning",
      "text": "Mantenha fora do alcance de criancas"
    },
    {
      "icon": "pregnant",
      "text": "Consulte seu medico antes de usar na gravidez"
    }
  ]
}
```

### 3.4 med_brand

Armazena marcas comerciais dos medicamentos.

```sql
CREATE TABLE medinfo.med_brand (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    medication_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    manufacturer VARCHAR(255),
    barcode VARCHAR(50),
    presentation VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_brand PRIMARY KEY (id),
    CONSTRAINT fk_brand_medication FOREIGN KEY (medication_id)
        REFERENCES medinfo.med_medication(id) ON DELETE CASCADE,
    CONSTRAINT uq_brand_barcode UNIQUE (barcode)
);

CREATE INDEX idx_brand_medication_id ON medinfo.med_brand(medication_id);
CREATE INDEX idx_brand_name ON medinfo.med_brand(name);
CREATE INDEX idx_brand_barcode ON medinfo.med_brand(barcode);
```

| Coluna | Tipo | Obrigatorio | Descricao |
|--------|------|-------------|-----------|
| id | UUID | Sim | Identificador unico |
| medication_id | UUID | Sim | FK para med_medication |
| name | VARCHAR(255) | Sim | Nome da marca |
| manufacturer | VARCHAR(255) | Nao | Fabricante |
| barcode | VARCHAR(50) | Nao | Codigo de barras (unico) |
| presentation | VARCHAR(100) | Nao | Apresentacao (ex: "caixa com 20 comprimidos") |

### 3.5 Calculadora de Conversao mg/ml

> **Nota:** A calculadora de conversao mg/ml **nao requer tabela no banco de dados**.
> E um calculo simples realizado no cliente (mobile) ou servidor (API).

**Formula:**
```
Volume (ml) = Dose desejada (mg) / Concentracao (mg/ml)
```

**Exemplo:**
```
Dose desejada: 500 mg
Concentracao: 50 mg/ml
Volume = 500 / 50 = 10 ml
```

O calculo e feito em tempo real, sem necessidade de persistencia.

### 3.6 med_interaction

Armazena interacoes medicamentosas.

```sql
CREATE TABLE medinfo.med_interaction (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    medication_a_id UUID NOT NULL,
    medication_b_id UUID NOT NULL,
    severity VARCHAR(20) NOT NULL,
    description TEXT NOT NULL,
    clinical_effects TEXT,
    recommendation TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_interaction PRIMARY KEY (id),
    CONSTRAINT fk_interaction_medication_a FOREIGN KEY (medication_a_id)
        REFERENCES medinfo.med_medication(id) ON DELETE CASCADE,
    CONSTRAINT fk_interaction_medication_b FOREIGN KEY (medication_b_id)
        REFERENCES medinfo.med_medication(id) ON DELETE CASCADE,
    CONSTRAINT ck_interaction_severity CHECK (
        severity IN ('MILD', 'MODERATE', 'SEVERE', 'CONTRAINDICATED')
    ),
    CONSTRAINT uq_interaction_pair UNIQUE (medication_a_id, medication_b_id)
);

CREATE INDEX idx_interaction_medication_a ON medinfo.med_interaction(medication_a_id);
CREATE INDEX idx_interaction_medication_b ON medinfo.med_interaction(medication_b_id);
```

### 3.7 med_favorite

Armazena medicamentos favoritos dos usuarios.

```sql
CREATE TABLE medinfo.med_favorite (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    medication_id UUID NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_favorite PRIMARY KEY (id),
    CONSTRAINT fk_favorite_user FOREIGN KEY (user_id)
        REFERENCES medinfo.med_user(id) ON DELETE CASCADE,
    CONSTRAINT fk_favorite_medication FOREIGN KEY (medication_id)
        REFERENCES medinfo.med_medication(id) ON DELETE CASCADE,
    CONSTRAINT uq_favorite_user_medication UNIQUE (user_id, medication_id)
);

CREATE INDEX idx_favorite_user_id ON medinfo.med_favorite(user_id);
```

### 3.8 med_payment

Armazena pagamentos realizados.

```sql
CREATE TABLE medinfo.med_payment (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    currency CHAR(3) DEFAULT 'BRL',
    status VARCHAR(20) NOT NULL,
    payment_method VARCHAR(30) NOT NULL,
    external_id VARCHAR(100),
    gateway VARCHAR(30),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_payment PRIMARY KEY (id),
    CONSTRAINT fk_payment_user FOREIGN KEY (user_id)
        REFERENCES medinfo.med_user(id) ON DELETE RESTRICT,
    CONSTRAINT ck_payment_status CHECK (
        status IN ('PENDING', 'APPROVED', 'REJECTED', 'REFUNDED', 'CANCELLED')
    ),
    CONSTRAINT ck_payment_method CHECK (
        payment_method IN ('CREDIT_CARD', 'DEBIT_CARD', 'PIX', 'PACKAGE')
    )
);

CREATE INDEX idx_payment_user_id ON medinfo.med_payment(user_id);
CREATE INDEX idx_payment_status ON medinfo.med_payment(status);
CREATE INDEX idx_payment_created_at ON medinfo.med_payment(created_at);
```

### 3.9 med_online_query

Armazena consultas online (pagas).

```sql
CREATE TABLE medinfo.med_online_query (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    medication_id UUID NOT NULL,
    payment_id UUID,
    query_type VARCHAR(20) DEFAULT 'SINGLE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_online_query PRIMARY KEY (id),
    CONSTRAINT fk_query_user FOREIGN KEY (user_id)
        REFERENCES medinfo.med_user(id) ON DELETE RESTRICT,
    CONSTRAINT fk_query_medication FOREIGN KEY (medication_id)
        REFERENCES medinfo.med_medication(id) ON DELETE RESTRICT,
    CONSTRAINT fk_query_payment FOREIGN KEY (payment_id)
        REFERENCES medinfo.med_payment(id) ON DELETE RESTRICT,
    CONSTRAINT ck_query_type CHECK (query_type IN ('SINGLE', 'PACKAGE'))
);

CREATE INDEX idx_query_user_id ON medinfo.med_online_query(user_id);
CREATE INDEX idx_query_created_at ON medinfo.med_online_query(created_at);
```

### 3.10 med_search_log

Armazena log de buscas para analytics.

```sql
CREATE TABLE medinfo.med_search_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID,
    query VARCHAR(255) NOT NULL,
    results_count INTEGER DEFAULT 0,
    source VARCHAR(10) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_search_log PRIMARY KEY (id),
    CONSTRAINT fk_search_user FOREIGN KEY (user_id)
        REFERENCES medinfo.med_user(id) ON DELETE SET NULL,
    CONSTRAINT ck_search_source CHECK (source IN ('LOCAL', 'REMOTE'))
);

CREATE INDEX idx_search_log_user_id ON medinfo.med_search_log(user_id);
CREATE INDEX idx_search_log_query ON medinfo.med_search_log(query);
CREATE INDEX idx_search_log_created_at ON medinfo.med_search_log(created_at);
```

## 4. Migrations

As migrations seguem o padrao Flyway:

```
migrations/
├── V001__create_schema.sql
├── V002__create_user_tables.sql
├── V003__create_medication_tables.sql
├── V004__create_payment_tables.sql
├── V005__create_indexes.sql
├── V006__seed_initial_medications.sql
└── V007__add_full_text_search.sql
```

### Exemplo V001__create_schema.sql

```sql
-- MedInfo Database Schema
-- Version: 1.0.0
-- Date: 2026-01-15

CREATE SCHEMA IF NOT EXISTS medinfo;

COMMENT ON SCHEMA medinfo IS 'Schema principal da aplicacao MedInfo';
```

## 5. Modelo Local (Room - Android)

O banco local do app Android usa Room com estrutura similar, mas otimizada para offline:

```kotlin
@Entity(tableName = "medication")
data class MedicationEntity(
    @PrimaryKey
    val id: String,

    @ColumnInfo(name = "commercial_name")
    val commercialName: String,

    @ColumnInfo(name = "active_ingredient")
    val activeIngredient: String,

    @ColumnInfo(name = "technical_info")
    val technicalInfo: String?, // JSON string

    @ColumnInfo(name = "simplified_info")
    val simplifiedInfo: String?, // JSON string

    @ColumnInfo(name = "synced_at")
    val syncedAt: Long
)

@Entity(
    tableName = "brand",
    foreignKeys = [
        ForeignKey(
            entity = MedicationEntity::class,
            parentColumns = ["id"],
            childColumns = ["medication_id"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class BrandEntity(
    @PrimaryKey
    val id: String,

    @ColumnInfo(name = "medication_id", index = true)
    val medicationId: String,

    val name: String,
    val manufacturer: String?,
    val barcode: String?
)
```

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
