# Setup de Desenvolvimento - MedInfo

## 1. Pre-requisitos

### 1.1 Software Necessario

| Software       | Versao    | Download                                       |
|----------------|-----------|------------------------------------------------|
| Node.js        | 22 LTS    | https://nodejs.org/                            |
| Android Studio | Hedgehog+ | https://developer.android.com/studio           |
| Docker Desktop | 24+       | https://www.docker.com/products/docker-desktop |
| Git            | 2.40+     | https://git-scm.com/                           |
| VS Code        | Latest    | https://code.visualstudio.com/                 |
| PostgreSQL     | 16        | Via Docker (recomendado)                       |

### 1.2 Extensoes VS Code Recomendadas

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "prisma.prisma",
    "orta.vscode-jest",
    "bradlc.vscode-tailwindcss",
    "ms-azuretools.vscode-docker"
  ]
}
```

### 1.3 Plugins Android Studio

- Kotlin (bundled)
- Android SDK 34
- Android Emulator
- Compose Multiplatform IDE Support

---

## 2. Configuracao do Ambiente

### 2.1 Clone do Repositorio

```bash
# Clone o repositorio
git clone https://github.com/seu-usuario/med-info.git
cd med-info

# Estrutura esperada
med-info/
├── backend/          # API NestJS
├── mobile/           # App Android (Kotlin)
├── database/         # Scripts SQL e migrations
├── docs/             # Documentacao
└── docker-compose.yml
```

### 2.2 Variaveis de Ambiente

#### Backend (.env)

Crie o arquivo `backend/.env`:

```text
# App
NODE_ENV=development
PORT=3000
API_PREFIX=api/v1

# Database
DATABASE_URL=postgresql://medinfo:medinfo123@localhost:5432/medinfo_dev
DATABASE_SCHEMA=medinfo

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=sua-chave-secreta-desenvolvimento-apenas
JWT_EXPIRES_IN=7d
JWT_REFRESH_SECRET=sua-chave-refresh-secreta
JWT_REFRESH_EXPIRES_IN=30d

# Payment Gateway (Sandbox)
PAYMENT_GATEWAY=stripe
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# Storage
STORAGE_PROVIDER=local
STORAGE_LOCAL_PATH=./uploads

# Logging
LOG_LEVEL=debug
```

#### Mobile (local.properties)

Crie/edite `mobile/local.properties`:

```text
# SDK Path (ajuste conforme seu sistema)
sdk.dir=/Users/seu-usuario/Library/Android/sdk
# ou Windows
# sdk.dir=C:\\Users\\seu-usuario\\AppData\\Local\\Android\\Sdk

# API URL para desenvolvimento
API_BASE_URL=http://10.0.2.2:3000/api/v1
```

---

## 3. Subindo os Servicos

### 3.1 Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: medinfo-postgres
    environment:
      POSTGRES_USER: medinfo
      POSTGRES_PASSWORD: medinfo123
      POSTGRES_DB: medinfo_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U medinfo -d medinfo_dev" ]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7.4-alpine
    container_name: medinfo-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 10s
      timeout: 5s
      retries: 5

  adminer:
    image: adminer
    container_name: medinfo-adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
```

```bash
# Subir servicos
docker-compose up -d

# Verificar status
docker-compose ps

# Ver logs
docker-compose logs -f postgres
```

### 3.2 Verificar Conexao

```bash
# Testar PostgreSQL
docker exec -it medinfo-postgres psql -U medinfo -d medinfo_dev -c "SELECT version();"

# Testar Redis
docker exec -it medinfo-redis redis-cli ping
```

---

## 4. Backend (NestJS)

### 4.1 Instalacao

```bash
cd backend

# Instalar dependencias
npm install

# Verificar instalacao
npm run --version
```

### 4.2 Estrutura do Projeto

```
backend/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   ├── common/
│   ├── config/
│   └── modules/
│       ├── auth/
│       ├── medication/
│       ├── user/
│       ├── dosage/
│       └── payment/
├── test/
├── prisma/              # ou TypeORM migrations
│   ├── schema.prisma
│   └── migrations/
├── .env
├── .env.example
├── nest-cli.json
├── package.json
└── tsconfig.json
```

### 4.3 Database Setup

```bash
# Usando Prisma
npx prisma generate
npx prisma migrate dev --name init
npx prisma db seed

# OU usando TypeORM
npm run typeorm migration:run
npm run seed
```

### 4.4 Executar Backend

```bash
# Desenvolvimento (watch mode)
npm run start:dev

# Debug mode
npm run start:debug

# Producao
npm run build
npm run start:prod
```

### 4.5 Verificar API

```bash
# Health check
curl http://localhost:3000/api/v1/health

# Swagger docs
open http://localhost:3000/api/docs
```

### 4.6 Testes

```bash
# Testes unitarios
npm run test

# Testes com watch
npm run test:watch

# Testes e2e
npm run test:e2e

# Coverage
npm run test:cov
```

---

## 5. Mobile (Android/Kotlin)

### 5.1 Abrir no Android Studio

1. Abra Android Studio
2. File > Open > Selecione a pasta `mobile/`
3. Aguarde o Gradle sync completar
4. Se solicitado, instale os SDKs necessarios

### 5.2 Estrutura do Projeto

```
mobile/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/medinfo/
│   │   │   │   ├── app/              # Application, DI
│   │   │   │   ├── data/             # Repositories impl
│   │   │   │   ├── domain/           # Models, UseCases
│   │   │   │   ├── presentation/     # UI, ViewModels
│   │   │   │   └── util/             # Utils
│   │   │   ├── res/
│   │   │   └── AndroidManifest.xml
│   │   ├── test/                     # Unit tests
│   │   └── androidTest/              # Instrumented tests
│   └── build.gradle.kts
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
└── local.properties
```

### 5.3 Configurar Emulador

1. Tools > Device Manager
2. Create Device
3. Selecione: Pixel 6 (ou similar)
4. System Image: API 34 (Android 14)
5. Finish

### 5.4 Executar App

```bash
# Via terminal
cd mobile
./gradlew assembleDebug
./gradlew installDebug

# Ou via Android Studio: Run > Run 'app'
```

### 5.5 Conectar com Backend Local

O emulador usa `10.0.2.2` para acessar o localhost da maquina host:

```kotlin
// Em BuildConfig ou arquivo de configuracao
object ApiConfig {
    val BASE_URL = if (BuildConfig.DEBUG) {
        "http://10.0.2.2:3000/api/v1/"
    } else {
        "https://api.medinfo.com.br/api/v1/"
    }
}
```

### 5.6 Testes

```bash
# Testes unitarios
./gradlew test

# Testes instrumentados (requer emulador/dispositivo)
./gradlew connectedAndroidTest

# Coverage
./gradlew jacocoTestReport
```

---

## 6. Banco de Dados

### 6.1 Acessar via Adminer

1. Abra http://localhost:8080
2. Sistema: PostgreSQL
3. Servidor: postgres (ou localhost se fora do Docker)
4. Usuario: medinfo
5. Senha: medinfo123
6. Base de dados: medinfo_dev

### 6.2 Acessar via CLI

```bash
# Conectar ao PostgreSQL
docker exec -it medinfo-postgres psql -U medinfo -d medinfo_dev

# Comandos uteis
\dt medinfo.*          # Listar tabelas
\d medinfo.med_user    # Descrever tabela
\q                     # Sair
```

### 6.3 Reset do Banco

```bash
# Via Docker
docker-compose down -v
docker-compose up -d
npm run prisma migrate dev  # ou typeorm migration:run

# Via SQL
DROP SCHEMA medinfo CASCADE;
CREATE SCHEMA medinfo;
```

---

## 7. Scripts Uteis

### 7.1 Package.json (Backend)

```json
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "build": "nest build",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "db:migrate": "prisma migrate dev",
    "db:seed": "prisma db seed",
    "db:studio": "prisma studio"
  }
}
```

### 7.2 Makefile (Opcional)

```yaml
.PHONY: up down logs backend mobile test clean

# Docker
up:
	docker-compose up -d

down:
	docker-compose down

logs:
	docker-compose logs -f

# Backend
backend:
	cd backend && npm run start:dev

backend-test:
	cd backend && npm run test

# Mobile
mobile:
	cd mobile && ./gradlew assembleDebug

mobile-test:
	cd mobile && ./gradlew test

# Limpeza
clean:
	docker-compose down -v
	cd backend && rm -rf node_modules dist
	cd mobile && ./gradlew clean
```

---

## 8. Troubleshooting

### 8.1 Problemas Comuns

| Problema                    | Solucao                                                     |
|-----------------------------|-------------------------------------------------------------|
| Porta 5432 em uso           | `docker-compose down` ou mude a porta no docker-compose.yml |
| Gradle sync falha           | File > Invalidate Caches > Restart                          |
| API nao conecta do emulador | Use `10.0.2.2` em vez de `localhost`                        |
| Migrations falham           | Verifique se PostgreSQL esta rodando                        |
| Redis connection refused    | Verifique se o container Redis esta up                      |

### 8.2 Logs

```bash
# Backend logs
cd backend && npm run start:dev 2>&1 | tee app.log

# Docker logs
docker-compose logs -f --tail=100

# Android logs
adb logcat | grep MedInfo
```

### 8.3 Limpar Cache

```bash
# Node modules
cd backend && rm -rf node_modules && npm install

# Gradle
cd mobile && ./gradlew clean

# Docker
docker system prune -a
```

---

## 9. Fluxo de Desenvolvimento

```
1. git checkout -b feature/MED-123-nova-feature
2. Fazer alteracoes
3. npm run test (backend) / ./gradlew test (mobile)
4. npm run lint (backend)
5. git add . && git commit -m "feat(scope): descricao"
6. git push origin feature/MED-123-nova-feature
7. Abrir Pull Request
```

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
