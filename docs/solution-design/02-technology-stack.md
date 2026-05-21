# 02 — Stack Tecnológica

> **Camada:** Foundation | **Depende de:** 01-solution-overview

## Backend

| Tecnologia | Versão | Papel |
|---|---|---|
| Java | 21 (LTS) | Linguagem principal |
| Spring Boot | 3.3.x | Framework de aplicação |
| Spring Web (MVC) | — | Controllers REST |
| Spring Data JPA | — | Acesso ao banco via ORM |
| Spring Security | — | Autenticação e autorização |
| Hibernate | 6.x | Implementação JPA |
| Flyway | — | Migrations de banco |
| Lombok | — | Redução de boilerplate |
| MapStruct | — | Mapeamento entre entidades e DTOs |
| SpringDoc OpenAPI | 2.x | Geração automática do Swagger UI |
| JUnit 5 | — | Testes unitários e de integração |
| Mockito | — | Mocks em testes |
| Testcontainers | — | PostgreSQL real em testes de integração |
| ArchUnit | 1.x | Testes de arquitetura — garante regras da Arquitetura Hexagonal |
| Maven | 3.9.x | Build e gerenciamento de dependências |

**Justificativas:**
- **Java 21**: Virtual threads (Project Loom) disponíveis; LTS com suporte até 2031; linguagem usada no SIAFI real.
- **Spring Boot 3.3**: Compatível com Java 21; suporte nativo a GraalVM se necessário no futuro.
- **Flyway**: Garante que o schema esteja sempre versionado e reproduzível; integração nativa com Spring Boot.
- **Testcontainers**: Testes de integração com PostgreSQL real, sem mocks de repositório — evita divergência entre teste e produção.

## Frontend

| Tecnologia | Versão | Papel |
|---|---|---|
| Node.js | 22 (LTS) | Runtime e tooling |
| React | 18.x | Biblioteca de UI |
| Vite | 5.x | Build tool e dev server |
| React Router | 6.x | Roteamento SPA |
| Axios | — | Cliente HTTP |
| TanStack Table | 8.x | Tabelas de dados com paginação e filtros |
| React Hook Form | 7.x | Formulários com validação |
| Zod | — | Validação de schemas (compartilhada com forms) |
| Jest + React Testing Library | — | Testes de componentes |
| ESLint + Prettier | — | Qualidade de código |

**Justificativas:**
- **React + Vite**: SPA pura adequada para sistema transacional interno (sem SEO necessário); Vite oferece HMR rápido em desenvolvimento.
- **TanStack Table**: Melhor opção React para tabelas complexas com filtros e paginação controlados pelo servidor.
- **React Hook Form + Zod**: Formulários extensos com validação tipada; performance superior a Formik.

**Nota sobre estilo**: O frontend usa o **Design System do Governo Federal (DSGov)** como referência visual. Implementado via CSS customizado com Tailwind CSS, respeitando a paleta e tipografia oficiais. Ver `07-frontend-design.md` para detalhes.

## Banco de Dados

| Tecnologia | Versão | Papel |
|---|---|---|
| PostgreSQL | 16 | Banco principal |
| pgcrypto | — | Extensão para funções criptográficas |

**Justificativas:**
- **PostgreSQL**: ACID completo; suporte a JSONB para armazenar dados de auditoria (`dados_antes`/`dados_depois`); índices parciais e expressões; open source.
- **JSONB para auditoria**: Permite armazenar o estado anterior e posterior de qualquer entidade sem schema fixo, facilitando a auditoria extensível.

## Infraestrutura e DevOps

| Tecnologia | Papel |
|---|---|
| Docker + Docker Compose | Ambiente local (banco + backend + frontend) |
| GitHub Actions | CI/CD: build, testes, lint |
| `.env` | Configuração de ambiente |

## Fora do Escopo (Fase 1)

| Tecnologia | Motivo da exclusão |
|---|---|
| Redis / Cache | Complexidade não justificada na Fase 1 |
| Kafka / RabbitMQ | Sem integração assíncrona no escopo atual |
| Kubernetes | Overhead operacional desnecessário para escopo educacional |
| GraphQL | REST é suficiente e mais simples para este domínio |
| Next.js / SSR | Não há requisito de SEO; SPA é suficiente |
