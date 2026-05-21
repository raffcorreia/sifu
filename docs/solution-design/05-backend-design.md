# 05 — Design do Backend

> **Camada:** Core Design | **Depende de:** 03-architecture, 04-database-design

## Estrutura de Pacotes

```
br.gov.sifu
│
├── auth/
│   ├── AuthController.java
│   ├── AuthService.java
│   ├── JwtService.java
│   ├── JwtFilter.java
│   └── dto/
│       ├── LoginRequest.java
│       ├── LoginResponse.java
│       └── PasswordResetRequest.java
│
├── managingunit/
│   ├── ManagingUnitController.java
│   ├── ManagingUnitService.java
│   ├── ManagingUnitRepository.java
│   ├── ManagingUnit.java              ← entidade JPA
│   └── dto/
│
├── classification/
│   ├── action/
│   │   ├── BudgetActionController.java
│   │   ├── BudgetActionService.java
│   │   ├── BudgetActionRepository.java
│   │   └── BudgetAction.java
│   ├── plan/
│   ├── expense/
│   ├── funding/
│   └── ptres/
│
├── allotment/
│   ├── BudgetAllotmentController.java
│   ├── BudgetAllotmentService.java
│   ├── BudgetAllotmentRepository.java
│   ├── BudgetAllotment.java
│   └── dto/
│
├── creditnote/
│   ├── CreditNoteController.java
│   ├── CreditNoteService.java
│   ├── CreditNoteRepository.java
│   ├── CreditNote.java
│   └── dto/
│
├── vendor/
│
├── commitment/
│   ├── CommitmentController.java
│   ├── CommitmentService.java
│   ├── CommitmentRepository.java
│   ├── Commitment.java
│   └── dto/
│
├── settlement/
│   ├── SettlementController.java
│   ├── SettlementService.java
│   ├── SettlementRepository.java
│   ├── Settlement.java
│   └── dto/
│
├── paymentorder/
│
├── report/
│   ├── ReportController.java          ← execução orçamentária e dashboard
│   └── ReportService.java
│
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   └── User.java
│
├── token/
│   ├── IntegrationTokenController.java
│   ├── IntegrationTokenService.java
│   └── IntegrationToken.java
│
├── audit/
│   ├── AuditController.java
│   ├── AuditService.java
│   ├── AuditInterceptor.java        ← AOP aspect
│   └── AuditEntry.java
│
└── common/
    ├── exception/
    │   ├── GlobalExceptionHandler.java  ← @ControllerAdvice
    │   ├── InsufficientBalanceException.java
    │   ├── EntityNotFoundException.java
    │   ├── InvalidStateTransitionException.java
    │   └── ...
    ├── pagination/
    │   └── PaginationUtils.java
    ├── security/
    │   ├── SecurityConfig.java
    │   └── SecurityContext.java       ← acesso ao usuário logado
    └── sequence/
        └── DocumentNumberGenerator.java
```

## Camadas e Responsabilidades

### Controller
- Deserializa e valida a requisição HTTP (via Bean Validation `@Valid`)
- Chama o Service
- Serializa a resposta como DTO
- Não contém lógica de negócio

### Service
- Toda a lógica de negócio e validações de domínio
- Coordena operações entre repositórios
- Lança exceções de domínio (`InsufficientBalanceException`, etc.)
- Anotado com `@Transactional` nas operações que alteram estado

### Repository
- Interfaces `JpaRepository` + queries JPQL/nativas para consultas complexas
- Named queries para consultas de execução orçamentária

### Entidade JPA
- Mapeamento direto com a tabela
- Validações de banco via `@Column(nullable = false)` etc.
- Sem lógica de negócio

## Tratamento de Erros

Todas as respostas de erro seguem o formato RFC 7807 (Problem Details):

```json
{
  "type": "https://sifu.gov.br/errors/insufficient-balance",
  "title": "Insufficient balance",
  "status": 422,
  "detail": "Saldo disponível da dotação 42 é R$ 1.000,00; valor solicitado: R$ 1.500,00",
  "instance": "/api/v1/commitments"
}
```

| Exceção | HTTP Status |
|---|---|
| `EntityNotFoundException` | 404 |
| `InsufficientBalanceException` | 422 |
| `InvalidStateTransitionException` | 422 |
| Violação de regra de negócio | 422 |
| Validação de campo | 400 |
| Não autenticado | 401 |
| Token expirado/revogado | 401 |
| Conta bloqueada | 423 |

## Auditoria via AOP

O `AuditInterceptor` usa Spring AOP para interceptar chamadas de service marcadas com `@Auditavel`:

```java
// Example usage
@Auditavel(operation = "ISSUE_COMMITMENT", entity = "Commitment")
public Commitment issue(CommitmentRequest request) { ... }
```

O interceptor captura automaticamente:
- Usuário logado (via `SecurityContext`)
- IP da requisição (via `HttpServletRequest`)
- Estado antes e depois (serializado em JSON)
- Timestamp

## Configuração Spring Security

```
Endpoints públicos:    POST /api/v1/auth/login
                       POST /api/v1/auth/recover-password
                       GET  /swagger-ui/**
                       GET  /api-docs/**
                       GET  /actuator/health

Endpoints autenticados: todos os demais /api/v1/**
```

Dois tipos de token aceitos pelo `JwtFilter`:
- **JWT de sessão**: emitido pelo login, expira em 8h, armazena `login` e `type=SESSION`
- **Token de integração**: emitido via `/api/v1/tokens`, expira conforme configurado, armazena `login` e `type=INTEGRATION`

## Paginação

Todas as listagens aceitam os parâmetros:

| Parâmetro | Padrão | Máximo |
|---|---|---|
| `page` | 0 | — |
| `size` | 20 | 100 |
| `sortBy` | depende do endpoint | — |
| `direction` | `ASC` | `ASC` ou `DESC` |

Resposta padrão de listagem:

```json
{
  "content": [...],
  "page": 0,
  "size": 20,
  "totalElements": 143,
  "totalPages": 8,
  "last": false
}
```
