# MedInfo - Aplicacao de Informacoes sobre Medicamentos

## Visao Geral

**MedInfo** e uma aplicacao mobile de saude para consulta de informacoes sobre medicamentos, com funcionalidades offline e online. O sistema oferece bulas digitais adaptativas, calculadora de dosagem para profissionais e um modelo freemium de monetizacao.

## Stack Tecnologica

| Camada | Tecnologia | Versao |
|--------|-----------|--------|
| Mobile | Kotlin + Jetpack Compose | Kotlin 2.0.x / SDK 34 |
| Backend | NestJS (Node.js + TypeScript) | Node 22 LTS / NestJS 10.x |
| Database | PostgreSQL | 16 |
| Cache | Redis | 7.4 |
| Arquitetura Mobile | MVVM | - |
| Arquitetura Backend | Clean Architecture | - |

## Publicos-Alvo

- **Profissionais de saude**: Medicos, farmaceuticos, enfermeiros
- **Estudantes**: Graduacao e pos-graduacao na area da saude
- **Publico geral**: Pessoas interessadas em informacoes sobre medicamentos

## Funcionalidades Principais

### Bula Digital Adaptativa
- Base de dados local com medicamentos comuns (offline)
- Minimo 3 marcas comerciais por medicamento
- **Visao Tecnica**: Informacoes farmacologicas completas para profissionais
- **Visao Simplificada**: Linguagem acessivel para publico geral
- Consulta online paga para medicamentos nao disponiveis localmente

### Calculadora de Dosagem
- Exclusiva para Profissionais e Estudantes validados
- Funciona offline apos validacao inicial
- Parametros: peso, altura, sexo, dosagem, outros clinicos
- Calculo automatico de dose recomendada

## Modelo de Monetizacao

| Tipo | Acesso | Custo |
|------|--------|-------|
| Gratuito | Medicamentos no banco local | R$ 0,00 |
| Pago | Consultas online expandidas | Por consulta |

## Estrutura do Projeto

```
med-info/
├── docs/                    # Documentacao tecnica
├── mobile/                  # App Android (Kotlin)
│   ├── app/
│   ├── data/               # Camada de dados
│   ├── domain/             # Regras de negocio
│   └── presentation/       # UI (MVVM)
├── backend/                 # API NestJS
│   ├── src/
│   │   ├── modules/        # Modulos da aplicacao
│   │   ├── common/         # Utilitarios compartilhados
│   │   └── config/         # Configuracoes
│   └── test/
└── database/               # Scripts e migrations
```

## Quick Start

### Pre-requisitos

- Android Studio Hedgehog+ (para mobile)
- Node.js 22 LTS
- PostgreSQL 16
- Docker e Docker Compose (recomendado)

### Executar Backend

```bash
cd backend
npm install
npm run start:dev
```

### Executar Mobile

```bash
cd mobile
./gradlew assembleDebug
```

## Documentacao

| Documento                                  | Descricao |
|--------------------------------------------|-----------|
| [01-VISION.md](01-VISION.md)               | Visao estrategica do produto |
| [02-REQUIREMENTS.md](02-REQUIREMENTS.md)   | Requisitos funcionais e nao-funcionais |
| [03-ARCHITECTURE.md](03-ARCHITECTURE.md)   | Arquitetura e decisoes tecnicas |
| [04-DATABASE.md](04-DATABASE.md)           | Modelo de dados |
| [05-API.md](05-API.md)                     | Documentacao de endpoints |
| [06-SETUP-DEV.md](06-SETUP-DEV.md)         | Configuracao do ambiente |
| [07-CONVENTIONS.md](07-CONVENTIONS.md)     | Convencoes de codigo |
| [08-DOMAIN-MODELS.md](08-DOMAIN-MODELS.md) | Modelos de dominio |
| [09-SECURITY.md](09-SECURITY.md)           | Seguranca e LGPD |
| [10-MONETIZATION.md](10-MONETIZATION.md)   | Modelo de monetizacao |

## Status do Projeto

- **Versao**: 1.0.0-SNAPSHOT
- **Status**: Em Desenvolvimento
- **Data Inicio**: Janeiro 2026

## Licenca

Proprietario - Todos os direitos reservados.

---

**Versao do Documento**: 1.0
**Ultima Atualizacao**: 15/01/2026
