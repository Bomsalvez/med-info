# Convencoes de Codigo - MedInfo

## 1. Principios Gerais

- **Codigo**: SEMPRE em ingles (variaveis, funcoes, classes, comentarios em codigo)
- **Documentacao**: SEMPRE em portugues (pt-BR)
- **Commits**: Conventional Commits em ingles
- **Nomenclatura**: Consistente entre camadas e projetos

---

## 2. Kotlin (Mobile)

### 2.1 Nomenclatura

| Elemento | Convencao | Exemplo |
|----------|-----------|---------|
| Classes | PascalCase | `MedicationRepository` |
| Interfaces | PascalCase (sem prefixo I) | `MedicationDataSource` |
| Funcoes | camelCase | `getMedicationById()` |
| Variaveis | camelCase | `medicationName` |
| Constantes | UPPER_SNAKE_CASE | `MAX_SEARCH_RESULTS` |
| Packages | lowercase | `com.medinfo.domain.model` |
| Arquivos | PascalCase (mesmo nome da classe) | `MedicationRepository.kt` |

### 2.2 Estrutura de Packages

```
com.medinfo/
├── app/                          # Application class, DI setup
├── data/                         # Camada de dados
│   ├── local/                    # Room, DataStore
│   │   ├── dao/                  # Data Access Objects
│   │   ├── entity/               # Entidades Room
│   │   └── mapper/               # Entity <-> Domain mappers
│   ├── remote/                   # Retrofit, APIs
│   │   ├── api/                  # Interface definitions
│   │   ├── dto/                  # Data Transfer Objects
│   │   └── mapper/               # DTO <-> Domain mappers
│   └── repository/               # Repository implementations
├── domain/                       # Camada de dominio (pura)
│   ├── model/                    # Modelos de dominio
│   ├── repository/               # Repository interfaces
│   └── usecase/                  # Casos de uso
├── presentation/                 # Camada de apresentacao (MVVM)
│   ├── ui/                       # Composables
│   │   ├── components/           # Componentes reutilizaveis
│   │   ├── screens/              # Telas
│   │   └── theme/                # Tema Material
│   └── viewmodel/                # ViewModels
└── util/                         # Utilitarios
```

### 2.3 Padroes de Codigo Kotlin

```kotlin
// Classe de dados (Domain Model)
data class Medication(
    val id: Long,
    val commercialName: String,
    val activeIngredient: String,
    val brands: List<Brand>,
    val technicalInfo: TechnicalInfo?,
    val simplifiedInfo: SimplifiedInfo?
)

// Sealed class para estados de UI
sealed class MedicationUiState {
    object Loading : MedicationUiState()
    data class Success(val medication: Medication) : MedicationUiState()
    data class Error(val message: String) : MedicationUiState()
}

// Repository interface (Domain layer)
interface MedicationRepository {
    suspend fun getMedicationById(id: Long): Result<Medication>
    suspend fun searchMedications(query: String): Result<List<Medication>>
    fun observeFavorites(): Flow<List<Medication>>
}

// UseCase
class GetMedicationByIdUseCase(
    private val repository: MedicationRepository
) {
    suspend operator fun invoke(id: Long): Result<Medication> {
        return repository.getMedicationById(id)
    }
}

// ViewModel
@HiltViewModel
class MedicationDetailViewModel @Inject constructor(
    private val getMedicationById: GetMedicationByIdUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<MedicationUiState>(MedicationUiState.Loading)
    val uiState: StateFlow<MedicationUiState> = _uiState.asStateFlow()

    fun loadMedication(id: Long) {
        viewModelScope.launch {
            getMedicationById(id)
                .onSuccess { _uiState.value = MedicationUiState.Success(it) }
                .onFailure { _uiState.value = MedicationUiState.Error(it.message ?: "Unknown error") }
        }
    }
}

// Composable
@Composable
fun MedicationDetailScreen(
    viewModel: MedicationDetailViewModel = hiltViewModel(),
    onNavigateBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    when (val state = uiState) {
        is MedicationUiState.Loading -> LoadingIndicator()
        is MedicationUiState.Success -> MedicationContent(state.medication)
        is MedicationUiState.Error -> ErrorMessage(state.message)
    }
}
```

---

## 3. TypeScript (Backend - NestJS)

### 3.1 Nomenclatura

| Elemento | Convencao | Exemplo |
|----------|-----------|---------|
| Classes | PascalCase | `MedicationService` |
| Interfaces | PascalCase (sem prefixo I) | `MedicationResponse` |
| Types | PascalCase | `UserRole` |
| Funcoes | camelCase | `getMedicationById()` |
| Variaveis | camelCase | `medicationName` |
| Constantes | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| Enums | PascalCase (valores UPPER_SNAKE_CASE) | `UserRole.PROFESSIONAL` |
| Arquivos | kebab-case | `medication.service.ts` |
| Diretorios | kebab-case | `medication-module/` |

### 3.2 Estrutura de Diretorios

```
src/
├── app.module.ts                 # Modulo raiz
├── main.ts                       # Bootstrap
├── common/                       # Utilitarios compartilhados
│   ├── decorators/               # Decorators customizados
│   ├── filters/                  # Exception filters
│   ├── guards/                   # Auth guards
│   ├── interceptors/             # Interceptors
│   ├── pipes/                    # Validation pipes
│   └── utils/                    # Funcoes utilitarias
├── config/                       # Configuracoes
│   ├── database.config.ts
│   ├── auth.config.ts
│   └── app.config.ts
└── modules/                      # Modulos de dominio
    ├── auth/                     # Autenticacao
    │   ├── dto/
    │   ├── entities/
    │   ├── auth.controller.ts
    │   ├── auth.service.ts
    │   └── auth.module.ts
    ├── medication/               # Medicamentos
    │   ├── dto/
    │   ├── entities/
    │   ├── medication.controller.ts
    │   ├── medication.service.ts
    │   └── medication.module.ts
    └── user/                     # Usuarios
        ├── dto/
        ├── entities/
        ├── user.controller.ts
        ├── user.service.ts
        └── user.module.ts
```

### 3.3 Padroes de Codigo TypeScript

```typescript
// Entity
@Entity('med_medication')
export class Medication {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ name: 'commercial_name' })
  commercialName: string;

  @Column({ name: 'active_ingredient' })
  activeIngredient: string;

  @Column({ name: 'created_at' })
  createdAt: Date;

  @OneToMany(() => Brand, (brand) => brand.medication)
  brands: Brand[];
}

// DTO
export class CreateMedicationDto {
  @IsString()
  @IsNotEmpty()
  @ApiProperty({ description: 'Nome comercial do medicamento' })
  commercialName: string;

  @IsString()
  @IsNotEmpty()
  @ApiProperty({ description: 'Principio ativo' })
  activeIngredient: string;
}

// Response DTO
export class MedicationResponseDto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  commercialName: string;

  @ApiProperty()
  activeIngredient: string;

  @ApiProperty({ type: [BrandResponseDto] })
  brands: BrandResponseDto[];
}

// Service
@Injectable()
export class MedicationService {
  constructor(
    @InjectRepository(Medication)
    private readonly medicationRepository: Repository<Medication>,
  ) {}

  async findById(id: string): Promise<Medication> {
    const medication = await this.medicationRepository.findOne({
      where: { id },
      relations: ['brands'],
    });

    if (!medication) {
      throw new NotFoundException(`Medication with ID ${id} not found`);
    }

    return medication;
  }

  async search(query: string, page: number, limit: number): Promise<PaginatedResult<Medication>> {
    const [items, total] = await this.medicationRepository.findAndCount({
      where: [
        { commercialName: ILike(`%${query}%`) },
        { activeIngredient: ILike(`%${query}%`) },
      ],
      skip: (page - 1) * limit,
      take: limit,
    });

    return { items, total, page, limit };
  }
}

// Controller
@Controller('medications')
@ApiTags('Medications')
export class MedicationController {
  constructor(private readonly medicationService: MedicationService) {}

  @Get(':id')
  @ApiOperation({ summary: 'Buscar medicamento por ID' })
  @ApiResponse({ status: 200, type: MedicationResponseDto })
  @ApiResponse({ status: 404, description: 'Medicamento nao encontrado' })
  async findById(@Param('id', ParseUUIDPipe) id: string): Promise<MedicationResponseDto> {
    const medication = await this.medicationService.findById(id);
    return this.mapToResponse(medication);
  }

  @Get()
  @ApiOperation({ summary: 'Buscar medicamentos' })
  @ApiQuery({ name: 'query', required: false })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  async search(
    @Query('query') query: string = '',
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ): Promise<PaginatedResult<MedicationResponseDto>> {
    return this.medicationService.search(query, page, limit);
  }

  private mapToResponse(medication: Medication): MedicationResponseDto {
    return {
      id: medication.id,
      commercialName: medication.commercialName,
      activeIngredient: medication.activeIngredient,
      brands: medication.brands.map((b) => ({
        id: b.id,
        name: b.name,
        manufacturer: b.manufacturer,
      })),
    };
  }
}
```

---

## 4. Banco de Dados (PostgreSQL)

### 4.1 Nomenclatura

| Elemento | Convencao | Exemplo |
|----------|-----------|---------|
| Tabelas | `med_{nome}` (singular, snake_case) | `med_medication` |
| Colunas | `{nome}` (snake_case) | `commercial_name` |
| Primary Keys | `id` (UUID ou SERIAL) | `id` |
| Foreign Keys | `{tabela_ref}_id` | `medication_id` |
| Indices | `idx_{tabela}_{coluna}` | `idx_medication_commercial_name` |
| Constraints PK | `pk_{tabela}` | `pk_medication` |
| Constraints FK | `fk_{tabela}_{tabela_ref}` | `fk_brand_medication` |
| Constraints Unique | `uq_{tabela}_{coluna}` | `uq_user_email` |
| Constraints Check | `ck_{tabela}_{descricao}` | `ck_user_role_valid` |

### 4.2 Exemplo de DDL

```sql
-- Schema
CREATE SCHEMA IF NOT EXISTS medinfo;

-- Tabela de usuarios
CREATE TABLE medinfo.med_user (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(150) NOT NULL,
    role VARCHAR(20) NOT NULL,
    professional_council VARCHAR(10),
    council_number VARCHAR(20),
    council_state CHAR(2),
    is_validated BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_user PRIMARY KEY (id),
    CONSTRAINT uq_user_email UNIQUE (email),
    CONSTRAINT ck_user_role_valid CHECK (role IN ('PROFESSIONAL', 'STUDENT', 'GENERAL'))
);

-- Tabela de medicamentos
CREATE TABLE medinfo.med_medication (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    commercial_name VARCHAR(255) NOT NULL,
    active_ingredient VARCHAR(255) NOT NULL,
    is_available_offline BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_medication PRIMARY KEY (id)
);

-- Tabela de marcas
CREATE TABLE medinfo.med_brand (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    medication_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    manufacturer VARCHAR(255),

    CONSTRAINT pk_brand PRIMARY KEY (id),
    CONSTRAINT fk_brand_medication FOREIGN KEY (medication_id)
        REFERENCES medinfo.med_medication(id) ON DELETE CASCADE
);

-- Indices
CREATE INDEX idx_medication_commercial_name ON medinfo.med_medication(commercial_name);
CREATE INDEX idx_medication_active_ingredient ON medinfo.med_medication(active_ingredient);
CREATE INDEX idx_brand_medication_id ON medinfo.med_brand(medication_id);
CREATE INDEX idx_user_email ON medinfo.med_user(email);
```

---

## 5. API REST

### 5.1 Convencoes de Endpoints

| Metodo | Padrao | Exemplo |
|--------|--------|---------|
| GET (lista) | `/resources` | `GET /medications` |
| GET (item) | `/resources/:id` | `GET /medications/123` |
| POST | `/resources` | `POST /medications` |
| PUT | `/resources/:id` | `PUT /medications/123` |
| PATCH | `/resources/:id` | `PATCH /medications/123` |
| DELETE | `/resources/:id` | `DELETE /medications/123` |

### 5.2 Versionamento

- Versao na URL: `/api/v1/medications`
- Header Accept: `Accept: application/vnd.medinfo.v1+json`

### 5.3 Padrao de Resposta

```typescript
// Sucesso
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2026-01-15T10:00:00Z"
  }
}

// Erro
{
  "success": false,
  "error": {
    "code": "MEDICATION_NOT_FOUND",
    "message": "Medicamento nao encontrado",
    "details": []
  },
  "meta": {
    "timestamp": "2026-01-15T10:00:00Z"
  }
}

// Lista paginada
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
    "timestamp": "2026-01-15T10:00:00Z"
  }
}
```

---

## 6. Git

### 6.1 Conventional Commits

```
{type}({scope}): {message}

Tipos:
- feat: Nova funcionalidade
- fix: Correcao de bug
- docs: Documentacao
- style: Formatacao (sem mudanca de codigo)
- refactor: Refatoracao
- test: Testes
- chore: Tarefas de build/config
- perf: Melhoria de performance
- ci: CI/CD

Scopes:
- mobile: App Kotlin
- backend: API NestJS
- database: Migrations/Scripts
- docs: Documentacao
- infra: Infraestrutura
```

### 6.2 Exemplos

```bash
feat(mobile): add medication search with autocomplete
fix(backend): resolve null pointer in dosage calculator
docs(api): update swagger documentation for medications endpoint
test(mobile): add unit tests for MedicationRepository
chore(deps): upgrade NestJS to 10.3.0
```

### 6.3 Branches

| Tipo | Padrao | Exemplo |
|------|--------|---------|
| Feature | `feature/{ticket}-{descricao}` | `feature/MED-123-medication-search` |
| Bugfix | `bugfix/{ticket}-{descricao}` | `bugfix/MED-456-fix-dosage-calc` |
| Hotfix | `hotfix/{ticket}-{descricao}` | `hotfix/MED-789-critical-auth-fix` |
| Release | `release/v{versao}` | `release/v1.0.0` |

---

## 7. Testes

### 7.1 Kotlin (Mobile)

| Tipo | Sufixo | Exemplo |
|------|--------|---------|
| Unit Test | `Test` | `MedicationRepositoryTest` |
| Integration Test | `IntegrationTest` | `MedicationApiIntegrationTest` |
| UI Test | `UiTest` | `MedicationSearchUiTest` |

### 7.2 TypeScript (Backend)

| Tipo | Sufixo | Exemplo |
|------|--------|---------|
| Unit Test | `.spec.ts` | `medication.service.spec.ts` |
| E2E Test | `.e2e-spec.ts` | `medication.e2e-spec.ts` |

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
