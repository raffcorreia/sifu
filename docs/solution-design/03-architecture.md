# 03 — Arquitetura

> **Camada:** Foundation | **Depende de:** 02-technology-stack

## Estilo Arquitetural

**Monolito modular** com separação clara por domínio de negócio. Os módulos se comunicam internamente via chamadas de serviço Java (não via HTTP), mas cada módulo tem seu próprio pacote de controller, service, repository e DTOs.

Escolha justificada: o escopo educacional não justifica a complexidade operacional de microsserviços. O monolito modular permite evolução para microsserviços no futuro sem rearchitetura completa do domínio.

## Diagrama de Componentes

```
┌──────────────────────────────────────────────────────────────────┐
│                        Navegador Web                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                  React SPA (Vite/Node)                     │  │
│  │  ┌────────────┐  ┌─────────────────┐  ┌────────────────┐  │  │
│  │  │   Sidebar  │  │  Formulários    │  │ Tabelas /      │  │  │
│  │  │   (Menu)   │  │  (RHF + Zod)   │  │ Dashboard      │  │  │
│  │  └────────────┘  └─────────────────┘  └────────────────┘  │  │
│  └────────────────────────────┬───────────────────────────────┘  │
└───────────────────────────────│──────────────────────────────────┘
                                │ HTTPS / REST+JSON
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                   Spring Boot (Java 21)                          │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │              Spring Security (JwtFilter)                │    │
│  └──────────────────────────────────────────────────────────┘    │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────┐    │
│  │  Controllers    │  │    Services      │  │ Repositories │    │
│  │  (REST /api/v1) │→ │  (Regras de      │→ │ (Spring Data │    │
│  │  + Swagger UI   │  │   Negócio)       │  │  JPA)        │    │
│  └─────────────────┘  └──────────────────┘  └──────┬───────┘    │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │         AuditInterceptor (AOP)                          │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
                                │ JDBC / Hibernate
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                      PostgreSQL 16                               │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────────────┐ │
│  │  Orçamento   │  │    Execução      │  │  Segurança /       │ │
│  │  (dotação,   │  │  (ne, nl, ob,    │  │  Auditoria         │ │
│  │   nc, class.)│  │   fornecedor)    │  │  (usuario, token,  │ │
│  └──────────────┘  └──────────────────┘  │   audit_log)       │ │
│                                          └────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

## Módulos do Backend

| Módulo | Pacote | Responsabilidade |
|---|---|---|
| Autenticação | `auth` | Login, logout, recuperação de senha, emissão JWT |
| Unidade Gestora | `managingunit` | CRUD de UGs e hierarquia |
| Classificações | `classification` | AcaoOrcamentaria, PlanoInterno, ND, FonteRecurso, PTRES |
| Dotação | `allotment` | Dotações orçamentárias e crédito suplementar |
| Nota de Crédito | `creditnote` | Emissão, aprovação, cancelamento e estorno de NC |
| Nota de Empenho | `commitment` | Emissão, reforço e anulação de NE |
| Liquidação | `settlement` | Registro e estorno de NL |
| Ordem Bancária | `paymentorder` | Emissão, processamento e cancelamento de OB |
| Fornecedor | `vendor` | CRUD de fornecedores |
| Consulta | `query` | Execução orçamentária e dashboard |
| Usuário | `user` | Gestão de usuários e senhas |
| Token | `token` | Geração, listagem e revogação de tokens de integração |
| Auditoria | `auditlog` | Consulta de logs de auditoria |
| Comum | `common` | Exceções, paginação, segurança transversal |

## Fluxo de Dados — Emissão de NE (exemplo)

```
Usuário
  │ POST /api/v1/commitments
  ▼
CommitmentController
  │ valida JWT via Spring Security
  │ deserializa DTO
  ▼
CommitmentService
  │ valida saldo da dotação (AllotmentService)
  │ gera número do empenho
  │ persiste Commitment
  │ AuditInterceptor registra operação (AOP)
  ▼
CommitmentRepository (JPA)
  │
  ▼
PostgreSQL
  │ transação ACID
  ▼
CommitmentController
  │ retorna 201 Created + DTO de resposta
  ▼
Usuário
```

## Fluxo de Autenticação

```
1. POST /api/v1/auth/login { login, senha }
2. Spring Security → AuthService → valida credenciais no banco
3. Emite JWT (accessToken, 8h) + registra na auditoria
4. Cliente armazena JWT no localStorage/memória
5. Cada requisição subsequente inclui: Authorization: Bearer <token>
6. JwtFilter valida assinatura + expiração em cada request

Para API externa:
1. POST /api/v1/tokens → gera token de integração (longo prazo)
2. Cliente usa: Authorization: Bearer <integration-token>
3. JwtFilter detecta tipo "INTEGRATION" e aplica validação própria
```

## Considerações de Performance

- Consultas de execução orçamentária usam views materializadas ou queries otimizadas com índices compostos.
- Paginação server-side obrigatória em todas as listagens (padrão: 20 itens/página, máximo: 100).
- Campos de busca frequente (numero_empenho, cnpj, codigo_ug) indexados no banco.
- Resposta esperada < 2s para volumes de dados médios conforme RNF.
