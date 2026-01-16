# Documentacao da API - MedInfo

## 1. Visao Geral

A API do MedInfo segue o padrao REST e utiliza JSON para comunicacao. Todas as rotas sao prefixadas com `/api/v1`.

### Base URL

| Ambiente | URL |
|----------|-----|
| Desenvolvimento | `http://localhost:3000/api/v1` |
| Staging | `https://staging-api.medinfo.com.br/api/v1` |
| Producao | `https://api.medinfo.com.br/api/v1` |

### Autenticacao

A API utiliza JWT (JSON Web Tokens) para autenticacao:

```
Authorization: Bearer <access_token>
```

## 2. Padrao de Resposta

### Sucesso

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

### Erro

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Descricao do erro",
    "details": [
      {
        "field": "email",
        "message": "Email invalido"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

### Lista Paginada

```json
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    },
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

## 3. Codigos de Erro

| Codigo | HTTP Status | Descricao |
|--------|-------------|-----------|
| VALIDATION_ERROR | 400 | Dados invalidos |
| UNAUTHORIZED | 401 | Token ausente ou invalido |
| FORBIDDEN | 403 | Acesso negado |
| NOT_FOUND | 404 | Recurso nao encontrado |
| CONFLICT | 409 | Conflito (ex: email duplicado) |
| PAYMENT_REQUIRED | 402 | Pagamento necessario |
| RATE_LIMIT_EXCEEDED | 429 | Limite de requisicoes excedido |
| INTERNAL_ERROR | 500 | Erro interno do servidor |

---

## 4. Endpoints

### 4.1 Autenticacao

#### POST /auth/register

Registra um novo usuario.

**Request Body:**

```json
{
  "email": "usuario@email.com",
  "password": "SenhaForte123!",
  "name": "Nome Completo",
  "role": "GENERAL"
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "usuario@email.com",
    "name": "Nome Completo",
    "role": "GENERAL",
    "isValidated": false,
    "createdAt": "2026-01-15T10:30:00Z"
  }
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| VALIDATION_ERROR | Campos invalidos |
| CONFLICT | Email ja cadastrado |

---

#### POST /auth/login

Autentica um usuario.

**Request Body:**

```json
{
  "email": "usuario@email.com",
  "password": "SenhaForte123!"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 604800,
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "usuario@email.com",
      "name": "Nome Completo",
      "role": "PROFESSIONAL",
      "isValidated": true
    }
  }
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| UNAUTHORIZED | Credenciais invalidas |

---

#### POST /auth/refresh

Renova o token de acesso.

**Request Body:**

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 604800
  }
}
```

---

### 4.2 Usuarios

#### GET /users/me

Retorna dados do usuario autenticado.

**Headers:** `Authorization: Bearer <token>`

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "medico@email.com",
    "name": "Dr. Joao Silva",
    "role": "PROFESSIONAL",
    "isValidated": true,
    "validation": {
      "councilType": "CRM",
      "councilNumber": "123456",
      "councilState": "SP",
      "status": "APPROVED",
      "validatedAt": "2026-01-10T14:00:00Z"
    },
    "createdAt": "2026-01-05T10:00:00Z"
  }
}
```

---

#### POST /users/validate-professional

Solicita validacao de profissional/estudante.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**

```json
{
  "councilType": "CRM",
  "councilNumber": "123456",
  "councilState": "SP",
  "documentUrl": "https://storage.medinfo.com.br/docs/abc123.pdf"
}
```

**Response (202 Accepted):**

```json
{
  "success": true,
  "data": {
    "validationId": "660e8400-e29b-41d4-a716-446655440001",
    "status": "PENDING",
    "message": "Solicitacao de validacao enviada. Voce sera notificado em ate 48 horas."
  }
}
```

---

### 4.3 Medicamentos

#### GET /medications

Busca medicamentos.

**Query Parameters:**

| Parametro | Tipo | Obrigatorio | Descricao |
|-----------|------|-------------|-----------|
| query | string | Nao | Termo de busca |
| page | number | Nao | Pagina (default: 1) |
| limit | number | Nao | Itens por pagina (default: 20, max: 100) |
| offlineOnly | boolean | Nao | Apenas medicamentos offline |

**Response (200 OK):**

```json
{
  "success": true,
  "data": [
    {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "commercialName": "Tylenol",
      "activeIngredient": "Paracetamol",
      "isOffline": true,
      "brands": [
        {"id": "...", "name": "Tylenol", "manufacturer": "Johnson & Johnson"},
        {"id": "...", "name": "Dorflex", "manufacturer": "Sanofi"},
        {"id": "...", "name": "Paracetamol Generico", "manufacturer": "EMS"}
      ]
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 500,
      "totalPages": 25
    }
  }
}
```

---

#### GET /medications/:id

Retorna detalhes de um medicamento.

**Headers:** `Authorization: Bearer <token>`

**Path Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| id | UUID | ID do medicamento |

**Query Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| view | string | `technical` ou `simplified` (default: baseado no perfil) |

**Response (200 OK) - Visao Simplificada:**

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "commercialName": "Tylenol",
    "activeIngredient": "Paracetamol",
    "isOffline": true,
    "brands": [...],
    "simplifiedInfo": {
      "whatIsIt": "Medicamento para aliviar dor e febre",
      "whatIsItFor": ["Dor de cabeca", "Dor no corpo", "Febre"],
      "howToUse": {
        "instructions": "Tomar com agua, de preferencia apos as refeicoes",
        "adults": "1 a 2 comprimidos a cada 4 ou 6 horas",
        "maxPerDay": "Nao ultrapassar 8 comprimidos em 24 horas"
      },
      "warnings": [...],
      "sideEffects": [...],
      "importantAlerts": [...]
    }
  }
}
```

**Response (200 OK) - Visao Tecnica (apenas Profissionais/Estudantes):**

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "commercialName": "Tylenol",
    "activeIngredient": "Paracetamol",
    "isOffline": true,
    "brands": [...],
    "technicalInfo": {
      "composition": {...},
      "pharmacokinetics": {...},
      "pharmacodynamics": {...},
      "indications": [...],
      "contraindications": [...],
      "dosage": {...},
      "interactions": [...],
      "adverseReactions": {...}
    }
  }
}
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| NOT_FOUND | Medicamento nao encontrado |
| FORBIDDEN | Visao tecnica requer perfil validado |
| PAYMENT_REQUIRED | Medicamento requer consulta online (paga) |

---

#### GET /medications/:id/interactions

Retorna interacoes medicamentosas.

**Headers:** `Authorization: Bearer <token>` (requer PROFESSIONAL ou STUDENT)

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "medicationId": "770e8400-e29b-41d4-a716-446655440002",
    "medicationName": "Paracetamol",
    "interactions": [
      {
        "medicationId": "880e8400-e29b-41d4-a716-446655440003",
        "medicationName": "Varfarina",
        "severity": "MODERATE",
        "description": "Paracetamol pode aumentar o efeito anticoagulante da varfarina",
        "clinicalEffects": "Aumento do INR e risco de sangramento",
        "recommendation": "Monitorar INR se uso prolongado de paracetamol"
      }
    ]
  }
}
```

---

### 4.4 Calculadora de Conversao mg/ml

Calculadora simples para converter miligramas (mg) em mililitros (ml).

**Formula:** `Volume (ml) = Dose (mg) / Concentracao (mg/ml)`

#### POST /dosage/convert

Converte dose em mg para volume em ml.

**Headers:** `Authorization: Bearer <token>` (requer PROFESSIONAL ou STUDENT validado)

**Request Body:**

```json
{
  "doseMg": 500,
  "concentrationMgMl": 50
}
```

| Campo | Tipo | Obrigatorio | Descricao |
|-------|------|-------------|-----------|
| doseMg | number | Sim | Dose desejada em miligramas (> 0) |
| concentrationMgMl | number | Sim | Concentracao do medicamento em mg/ml (> 0) |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "doseMg": 500,
    "concentrationMgMl": 50,
    "volumeMl": 10,
    "formula": "500 mg / 50 mg/ml = 10 ml"
  }
}
```

| Campo | Tipo | Descricao |
|-------|------|-----------|
| doseMg | number | Dose informada (mg) |
| concentrationMgMl | number | Concentracao informada (mg/ml) |
| volumeMl | number | Volume calculado (ml) |
| formula | string | Formula aplicada para exibicao |

**Exemplo de Uso:**

```bash
curl -X POST https://api.medinfo.com.br/api/v1/dosage/convert \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"doseMg": 500, "concentrationMgMl": 50}'
```

**Erros:**

| Codigo | Descricao |
|--------|-----------|
| FORBIDDEN | Requer perfil validado (PROFESSIONAL ou STUDENT) |
| VALIDATION_ERROR | Parametros invalidos (valores <= 0) |

---

### 4.5 Favoritos

#### GET /favorites

Lista medicamentos favoritos do usuario.

**Headers:** `Authorization: Bearer <token>`

**Response (200 OK):**

```json
{
  "success": true,
  "data": [
    {
      "id": "990e8400-e29b-41d4-a716-446655440004",
      "medication": {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "commercialName": "Tylenol",
        "activeIngredient": "Paracetamol"
      },
      "createdAt": "2026-01-10T15:00:00Z"
    }
  ]
}
```

---

#### POST /favorites

Adiciona medicamento aos favoritos.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**

```json
{
  "medicationId": "770e8400-e29b-41d4-a716-446655440002"
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "medicationId": "770e8400-e29b-41d4-a716-446655440002",
    "createdAt": "2026-01-15T10:30:00Z"
  }
}
```

---

#### DELETE /favorites/:medicationId

Remove medicamento dos favoritos.

**Headers:** `Authorization: Bearer <token>`

**Response (204 No Content)**

---

### 4.6 Pagamentos

#### POST /payments/checkout

Inicia processo de pagamento para consulta online.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**

```json
{
  "medicationId": "770e8400-e29b-41d4-a716-446655440002",
  "paymentMethod": "PIX"
}
```

**Response (200 OK) - PIX:**

```json
{
  "success": true,
  "data": {
    "paymentId": "aa0e8400-e29b-41d4-a716-446655440005",
    "amount": 2.99,
    "currency": "BRL",
    "status": "PENDING",
    "paymentMethod": "PIX",
    "pix": {
      "qrCode": "00020126580014br.gov.bcb.pix...",
      "qrCodeBase64": "data:image/png;base64,...",
      "copyPaste": "00020126580014br.gov.bcb.pix...",
      "expiresAt": "2026-01-15T11:00:00Z"
    }
  }
}
```

---

#### GET /payments/:id

Consulta status de um pagamento.

**Headers:** `Authorization: Bearer <token>`

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "paymentId": "aa0e8400-e29b-41d4-a716-446655440005",
    "amount": 2.99,
    "currency": "BRL",
    "status": "APPROVED",
    "paymentMethod": "PIX",
    "createdAt": "2026-01-15T10:30:00Z",
    "updatedAt": "2026-01-15T10:32:00Z"
  }
}
```

---

### 4.7 Sincronizacao (Mobile)

#### GET /sync/medications

Retorna medicamentos para sincronizacao local.

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**

| Parametro | Tipo | Descricao |
|-----------|------|-----------|
| since | ISO8601 | Timestamp da ultima sincronizacao |
| page | number | Pagina (para sincronizacao inicial) |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "medications": [...],
    "deleted": ["id1", "id2"],
    "lastSync": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "pagination": {
      "page": 1,
      "totalPages": 10,
      "hasMore": true
    }
  }
}
```

---

## 5. Rate Limiting

| Endpoint | Limite | Janela |
|----------|--------|--------|
| /auth/* | 10 req | 1 min |
| /medications/* | 100 req | 1 min |
| /dosage/* | 50 req | 1 min |
| /payments/* | 20 req | 1 min |
| /sync/* | 10 req | 1 min |

---

## 6. Swagger/OpenAPI

A documentacao interativa esta disponivel em:

- Desenvolvimento: `http://localhost:3000/api/docs`
- Staging: `https://staging-api.medinfo.com.br/api/docs`

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
