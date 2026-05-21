# Design de Solução — SIFU

> Este diretório contém o design técnico completo do SIFU. Os documentos são progressivos: cada camada depende das anteriores.

## Estrutura dos Documentos

| # | Documento | Camada | Descrição |
|---|---|---|---|
| 00 | index.md | — | Este índice |
| 01 | solution-overview.md | Foundation | Visão geral da solução, problema e escopo |
| 02 | technology-stack.md | Foundation | Stack tecnológica com justificativas |
| 03 | architecture.md | Foundation | Arquitetura, componentes e fluxo de dados |
| 04 | database-design.md | Core Design | Schema PostgreSQL, índices e migrações |
| 05 | backend-design.md | Core Design | Estrutura Spring Boot, camadas e padrões |
| 06 | api-design.md | Core Design | Endpoints REST, contratos e exemplos |
| 07 | frontend-design.md | Experience | Layout, telas, componentes e referências visuais SIAFI |
| 08 | security-design.md | Experience | Autenticação JWT, auditoria e controle de acesso |
| 09 | testing.md | Experience | Estratégia de testes, casos críticos e cobertura |
| 10 | integration.md | Operations | API para sistemas externos e tokens de integração |
| 11 | observability.md | Operations | Logging, health checks e métricas |
| 12 | deployment.md | Operations | Docker, variáveis de ambiente e CI/CD |

## Rastreamento de Requisitos

| Requisito | Documento(s) de Origem | Documentos de Design |
|---|---|---|
| RF01 — Login | requirements/03, requirements/05 | 07-frontend, 08-security |
| RF02 — Tokens JWT | requirements/03 | 08-security, 10-integration |
| RF03 — Classificações | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF04 — Dotações | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF05/06 — NC | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF07/08 — NE | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF09 — Liquidação | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF10 — OB | requirements/02, requirements/06 | 04-database, 05-backend, 06-api |
| RF11/12 — Consultas/Dashboard | requirements/03 | 06-api, 07-frontend |
| RF13–17 — API/Tokens | requirements/03, requirements/05 | 08-security, 10-integration |
| RF18/19 — API versionada + docs | requirements/03, requirements/05 | 06-api |
| RNF Segurança | requirements/05 | 08-security |
| RNF Auditoria | requirements/05 | 08-security, 04-database |
| RNF Performance | requirements/05 | 04-database, 11-observability |
| RNF Qualidade | requirements/05 | 09-testing |
| RNF Deploy | requirements/05 | 12-deployment |

## Dependências entre Documentos

```
01-solution-overview
       ↓
02-technology-stack
       ↓
03-architecture
    ↙     ↘
04-db   05-backend ── 06-api
           ↓
    07-frontend   08-security
           ↓
    09-testing
    10-integration
    11-observability
    12-deployment
```
