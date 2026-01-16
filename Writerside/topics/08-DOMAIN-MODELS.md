# Modelos de Dominio - MedInfo

## 1. Visao Geral

Este documento descreve as entidades de dominio do MedInfo, suas regras de negocio e relacionamentos.

---

## 2. Entidades Principais

### 2.1 User (Usuario)

Representa um usuario do sistema.

```
┌─────────────────────────────────────────────────────────────┐
│                          User                                │
├─────────────────────────────────────────────────────────────┤
│ + id: UUID                                                   │
│ + email: Email                                               │
│ + passwordHash: String                                       │
│ + name: String                                               │
│ + role: UserRole                                             │
│ + isValidated: Boolean                                       │
│ + lastLoginAt: DateTime?                                     │
│ + createdAt: DateTime                                        │
│ + updatedAt: DateTime                                        │
├─────────────────────────────────────────────────────────────┤
│ + validation: ProfessionalValidation?                        │
│ + favorites: List<Favorite>                                  │
│ + payments: List<Payment>                                    │
├─────────────────────────────────────────────────────────────┤
│ + canAccessTechnicalView(): Boolean                          │
│ + canUseDosageCalculator(): Boolean                          │
│ + isGeneral(): Boolean                                       │
│ + isProfessional(): Boolean                                  │
│ + isStudent(): Boolean                                       │
└─────────────────────────────────────────────────────────────┘
```

#### UserRole (Enum)

| Valor | Descricao | Acesso |
|-------|-----------|--------|
| GENERAL | Publico geral | Visao simplificada apenas |
| STUDENT | Estudante validado | Visao tecnica + calculadora |
| PROFESSIONAL | Profissional validado | Visao tecnica + calculadora |

#### Regras de Negocio {id="regras-de-negocio_1"}

| Regra | Descricao |
|-------|-----------|
| RN-U01 | Email deve ser unico no sistema |
| RN-U02 | Senha deve ter minimo 8 caracteres, 1 maiuscula, 1 numero |
| RN-U03 | Nome deve ter entre 3 e 150 caracteres |
| RN-U04 | `isValidated` so pode ser true para STUDENT ou PROFESSIONAL |
| RN-U05 | GENERAL nao pode acessar visao tecnica |
| RN-U06 | Apenas usuarios validados podem usar calculadora |

#### Codigo Kotlin (Domain) {id="codigo-kotlin-domain_1"}

```kotlin
data class User(
    val id: UserId,
    val email: Email,
    val name: String,
    val role: UserRole,
    val isValidated: Boolean,
    val lastLoginAt: Instant?,
    val createdAt: Instant,
    val updatedAt: Instant
) {
    fun canAccessTechnicalView(): Boolean =
        (role == UserRole.PROFESSIONAL || role == UserRole.STUDENT) && isValidated

    fun canUseDosageCalculator(): Boolean =
        canAccessTechnicalView()

    fun isGeneral(): Boolean = role == UserRole.GENERAL
    fun isProfessional(): Boolean = role == UserRole.PROFESSIONAL
    fun isStudent(): Boolean = role == UserRole.STUDENT
}

enum class UserRole {
    GENERAL,
    STUDENT,
    PROFESSIONAL
}

@JvmInline
value class UserId(val value: String)

@JvmInline
value class Email(val value: String) {
    init {
        require(value.matches(EMAIL_REGEX)) { "Email invalido" }
    }

    companion object {
        private val EMAIL_REGEX = Regex("^[A-Za-z0-9+_.-]+@(.+)$")
    }
}
```

---

### 2.2 ProfessionalValidation (Validacao Profissional)

Representa a validacao de credenciais de um profissional ou estudante.

```
┌─────────────────────────────────────────────────────────────┐
│                  ProfessionalValidation                      │
├─────────────────────────────────────────────────────────────┤
│ + id: UUID                                                   │
│ + userId: UUID                                               │
│ + councilType: CouncilType                                   │
│ + councilNumber: String                                      │
│ + councilState: BrazilianState                               │
│ + status: ValidationStatus                                   │
│ + validatedAt: DateTime?                                     │
│ + rejectedReason: String?                                    │
│ + documentUrl: String?                                       │
│ + createdAt: DateTime                                        │
├─────────────────────────────────────────────────────────────┤
│ + isPending(): Boolean                                       │
│ + isApproved(): Boolean                                      │
│ + isRejected(): Boolean                                      │
│ + approve(): ProfessionalValidation                          │
│ + reject(reason: String): ProfessionalValidation             │
└─────────────────────────────────────────────────────────────┘
```

#### CouncilType (Enum)

| Valor | Descricao |
|-------|-----------|
| CRM | Conselho Regional de Medicina |
| CRF | Conselho Regional de Farmacia |
| COREN | Conselho Regional de Enfermagem |
| CRO | Conselho Regional de Odontologia |
| CRBM | Conselho Regional de Biomedicina |
| STUDENT | Matricula estudantil |

#### ValidationStatus (Enum)

| Valor | Descricao |
|-------|-----------|
| PENDING | Aguardando validacao |
| APPROVED | Validacao aprovada |
| REJECTED | Validacao rejeitada |
| EXPIRED | Validacao expirada |

#### Regras de Negocio {id="regras-de-negocio_2"}

| Regra | Descricao |
|-------|-----------|
| RN-V01 | Numero do conselho deve seguir formato do respectivo orgao |
| RN-V02 | UF deve ser valida (26 estados + DF) |
| RN-V03 | Apenas PENDING pode transicionar para APPROVED ou REJECTED |
| RN-V04 | Ao aprovar, `validatedAt` deve ser preenchido |
| RN-V05 | Ao rejeitar, `rejectedReason` e obrigatorio |
| RN-V06 | Validacao expira em 1 ano (APPROVED -> EXPIRED) |

---

### 2.3 Medication (Medicamento)

Representa um medicamento no sistema.

```
┌─────────────────────────────────────────────────────────────┐
│                        Medication                            │
├─────────────────────────────────────────────────────────────┤
│ + id: UUID                                                   │
│ + commercialName: String                                     │
│ + activeIngredient: String                                   │
│ + description: String?                                       │
│ + isOffline: Boolean                                         │
│ + technicalInfo: TechnicalInfo?                              │
│ + simplifiedInfo: SimplifiedInfo?                            │
│ + createdAt: DateTime                                        │
│ + updatedAt: DateTime                                        │
├─────────────────────────────────────────────────────────────┤
│ + brands: List<Brand>                                        │
│ + interactions: List<Interaction>                            │
├─────────────────────────────────────────────────────────────┤
│ + isAvailableOffline(): Boolean                              │
│ + requiresPayment(): Boolean                                 │
│ + getInfoForUser(user: User): MedicationInfo                 │
│ + getBrandCount(): Int                                       │
└─────────────────────────────────────────────────────────────┘
```

#### Regras de Negocio

| Regra | Descricao |
|-------|-----------|
| RN-M01 | Todo medicamento deve ter pelo menos 1 marca |
| RN-M02 | Medicamentos offline devem ter no minimo 3 marcas |
| RN-M03 | `technicalInfo` obrigatorio para visao tecnica |
| RN-M04 | `simplifiedInfo` obrigatorio para visao simplificada |
| RN-M05 | `isOffline=false` implica consulta paga |

#### Codigo Kotlin (Domain)

```kotlin
data class Medication(
    val id: MedicationId,
    val commercialName: String,
    val activeIngredient: String,
    val description: String?,
    val isOffline: Boolean,
    val technicalInfo: TechnicalInfo?,
    val simplifiedInfo: SimplifiedInfo?,
    val brands: List<Brand>,
    val createdAt: Instant,
    val updatedAt: Instant
) {
    init {
        require(brands.isNotEmpty()) { "Medicamento deve ter pelo menos 1 marca" }
        if (isOffline) {
            require(brands.size >= 3) { "Medicamento offline deve ter pelo menos 3 marcas" }
        }
    }

    fun isAvailableOffline(): Boolean = isOffline

    fun requiresPayment(): Boolean = !isOffline

    fun getInfoForUser(user: User): MedicationInfo {
        return if (user.canAccessTechnicalView() && technicalInfo != null) {
            MedicationInfo.Technical(technicalInfo)
        } else {
            MedicationInfo.Simplified(simplifiedInfo ?: throw IllegalStateException("Simplified info required"))
        }
    }

    fun getBrandCount(): Int = brands.size
}

@JvmInline
value class MedicationId(val value: String)

sealed class MedicationInfo {
    data class Technical(val info: TechnicalInfo) : MedicationInfo()
    data class Simplified(val info: SimplifiedInfo) : MedicationInfo()
}
```

---

### 2.4 TechnicalInfo (Informacoes Tecnicas)

Value Object com informacoes farmacologicas completas.

```kotlin
data class TechnicalInfo(
    val composition: Composition,
    val pharmacokinetics: Pharmacokinetics,
    val pharmacodynamics: Pharmacodynamics,
    val indications: List<String>,
    val contraindications: List<String>,
    val dosage: DosageInfo,
    val interactions: List<DrugInteraction>,
    val adverseReactions: AdverseReactions
)

data class Composition(
    val activeIngredients: List<Ingredient>,
    val excipients: List<String>
)

data class Ingredient(
    val name: String,
    val amount: BigDecimal,
    val unit: String
)

data class Pharmacokinetics(
    val absorption: String,
    val distribution: String,
    val metabolism: String,
    val excretion: String
)

data class Pharmacodynamics(
    val mechanismOfAction: String,
    val therapeuticClass: String
)

data class DosageInfo(
    val adults: String,
    val children: String?,
    val elderly: String?,
    val maxDaily: String
)

data class DrugInteraction(
    val drug: String,
    val effect: String,
    val severity: InteractionSeverity
)

enum class InteractionSeverity {
    MILD, MODERATE, SEVERE, CONTRAINDICATED
}

data class AdverseReactions(
    val common: List<String>,
    val uncommon: List<String>,
    val rare: List<String>
)
```

---

### 2.5 SimplifiedInfo (Informacoes Simplificadas)

Value Object com informacoes acessiveis para publico geral.

```kotlin
data class SimplifiedInfo(
    val whatIsIt: String,
    val whatIsItFor: List<String>,
    val howToUse: HowToUse,
    val warnings: List<String>,
    val sideEffects: List<String>,
    val importantAlerts: List<Alert>
)

data class HowToUse(
    val instructions: String,
    val adults: String,
    val maxPerDay: String
)

data class Alert(
    val icon: AlertIcon,
    val text: String
)

enum class AlertIcon {
    WARNING,
    PREGNANT,
    CHILD,
    ELDERLY,
    DRIVING,
    ALCOHOL
}
```

---

### 2.6 Brand (Marca)

Representa uma marca comercial de um medicamento.

```kotlin
data class Brand(
    val id: BrandId,
    val name: String,
    val manufacturer: String?,
    val barcode: String?,
    val presentation: String?
)

@JvmInline
value class BrandId(val value: String)
```

#### Regras de Negocio {id="regras-de-negocio_3"}

| Regra | Descricao |
|-------|-----------|
| RN-B01 | Nome da marca e obrigatorio |
| RN-B02 | Barcode, se presente, deve ser unico |
| RN-B03 | Barcode deve ter 8 ou 13 digitos (EAN-8 ou EAN-13) |

---

### 2.7 DosageConverter (Conversor mg/ml)

Servico simples para converter miligramas (mg) em mililitros (ml).

> **Nota:** Este e um servico de dominio stateless, nao uma entidade.
> Nao requer persistencia no banco de dados.

**Formula:**
```
Volume (ml) = Dose (mg) / Concentracao (mg/ml)
```

```kotlin
/**
 * Servico de dominio para conversao de dosagem mg para ml.
 * Stateless - pode ser usado como singleton.
 */
object DosageConverter {

    /**
     * Converte dose em mg para volume em ml.
     *
     * @param doseMg Dose desejada em miligramas (deve ser > 0)
     * @param concentrationMgMl Concentracao do medicamento em mg/ml (deve ser > 0)
     * @return Resultado da conversao com volume e formula aplicada
     * @throws IllegalArgumentException se parametros forem invalidos
     */
    fun convert(doseMg: BigDecimal, concentrationMgMl: BigDecimal): ConversionResult {
        require(doseMg > BigDecimal.ZERO) { "Dose deve ser maior que zero" }
        require(concentrationMgMl > BigDecimal.ZERO) { "Concentracao deve ser maior que zero" }

        val volumeMl = doseMg.divide(concentrationMgMl, 2, RoundingMode.HALF_UP)

        return ConversionResult(
            doseMg = doseMg,
            concentrationMgMl = concentrationMgMl,
            volumeMl = volumeMl,
            formula = "${doseMg} mg / ${concentrationMgMl} mg/ml = ${volumeMl} ml"
        )
    }
}

/**
 * Resultado da conversao de dosagem.
 */
data class ConversionResult(
    val doseMg: BigDecimal,
    val concentrationMgMl: BigDecimal,
    val volumeMl: BigDecimal,
    val formula: String
)
```

#### Exemplo de Uso

```kotlin
// Converter 500mg com concentracao de 50mg/ml
val result = DosageConverter.convert(
    doseMg = BigDecimal("500"),
    concentrationMgMl = BigDecimal("50")
)

println(result.volumeMl)  // 10.00
println(result.formula)   // "500 mg / 50 mg/ml = 10.00 ml"
```

#### Regras de Negocio {id="regras-de-negocio_4"}

| Regra | Descricao |
|-------|-----------|
| RN-DC01 | Dose (mg) deve ser maior que zero |
| RN-DC02 | Concentracao (mg/ml) deve ser maior que zero |
| RN-DC03 | Resultado arredondado para 2 casas decimais |
| RN-DC04 | Acesso restrito a usuarios PROFESSIONAL ou STUDENT validados |

---

### 2.8 Payment (Pagamento)

Representa um pagamento no sistema.

```kotlin
data class Payment(
    val id: PaymentId,
    val userId: UserId,
    val amount: Money,
    val status: PaymentStatus,
    val paymentMethod: PaymentMethod,
    val externalId: String?,
    val gateway: String?,
    val createdAt: Instant,
    val updatedAt: Instant
) {
    fun isPending(): Boolean = status == PaymentStatus.PENDING
    fun isApproved(): Boolean = status == PaymentStatus.APPROVED
    fun canRefund(): Boolean = status == PaymentStatus.APPROVED
}

@JvmInline
value class PaymentId(val value: String)

data class Money(
    val amount: BigDecimal,
    val currency: Currency = Currency.BRL
) {
    init {
        require(amount >= BigDecimal.ZERO) { "Valor nao pode ser negativo" }
    }
}

enum class Currency { BRL, USD }

enum class PaymentStatus {
    PENDING,
    APPROVED,
    REJECTED,
    REFUNDED,
    CANCELLED
}

enum class PaymentMethod {
    CREDIT_CARD,
    DEBIT_CARD,
    PIX,
    PACKAGE
}
```

---

## 3. Agregados

### 3.1 User Aggregate

```
User (Root)
├── ProfessionalValidation (1:1)
├── Favorite (1:N)
└── Payment (1:N)
    └── OnlineQuery (1:N)
```

### 3.2 Medication Aggregate

```
Medication (Root)
├── Brand (1:N) - minimo 1, minimo 3 se offline
├── TechnicalInfo (1:1) - Value Object
└── SimplifiedInfo (1:1) - Value Object
```

> **Nota:** A calculadora de conversao mg/ml e um servico de dominio
> stateless (`DosageConverter`), nao faz parte de nenhum agregado.

---

## 4. Diagrama de Dominio

```
┌─────────────┐         ┌─────────────────────┐
│    User     │────────▶│ProfessionalValidation│
└─────────────┘         └─────────────────────┘
       │
       │ 1:N
       ▼
┌─────────────┐         ┌─────────────────────┐
│  Favorite   │────────▶│     Medication      │
└─────────────┘         └─────────────────────┘
       │                         │
       │                         │ 1:N
       ▼                         ▼
┌─────────────┐         ┌─────────────────────┐
│   Payment   │         │       Brand         │
└─────────────┘         └─────────────────────┘
       │                         │
       │ 1:N                     │
       ▼                         ▼
┌─────────────┐
│ OnlineQuery │
└─────────────┘

Nota: DosageConverter e um servico stateless,
nao uma entidade persistida no diagrama.
```

---

## 5. Invariantes de Dominio

| ID | Invariante |
|----|------------|
| INV-01 | Usuario GENERAL nunca pode ter `isValidated=true` |
| INV-02 | Medicamento offline deve ter >= 3 marcas |
| INV-03 | Barcode de marca deve ser unico no sistema |
| INV-04 | Validacao aprovada deve ter `validatedAt` preenchido |
| INV-05 | Validacao rejeitada deve ter `rejectedReason` preenchido |
| INV-06 | Pagamento aprovado so pode transicionar para REFUNDED |
| INV-07 | OnlineQuery deve ter Payment associado se medicamento nao e offline |

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
