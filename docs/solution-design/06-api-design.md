# 06 — API Design

> **Layer:** Core Design | **Depends on:** 05-backend-design

## OpenAPI Strategy

SIFU uses **code-first + annotation-driven** OpenAPI documentation via **SpringDoc OpenAPI 2.x**, which generates an OpenAPI 3.0.3 specification at runtime. The Swagger UI is the primary developer-facing API reference.

| Endpoint | Description |
|---|---|
| `GET /swagger-ui.html` | Interactive Swagger UI |
| `GET /api-docs` | Raw OpenAPI 3.0.3 JSON |
| `GET /api-docs.yaml` | Raw OpenAPI 3.0.3 YAML |

All documentation is **annotated at the source** — no separate YAML files to keep in sync with the code. The spec is always correct by construction.

---

## SpringDoc Configuration

### OpenAPI Bean (`infrastructure/config/OpenApiConfig.java`)

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI sifuOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("SIFU — Sistema Integrado Financeiro Unificado")
                .version("1.0.0")
                .description("""
                    API REST para gestão do ciclo completo da despesa pública federal.
                    Cobre o fluxo: Dotação → Nota de Crédito → Nota de Empenho
                    → Liquidação → Ordem Bancária.
                    """)
                .contact(new Contact()
                    .name("Equipe SIFU")
                    .email("sifu@gov.br")))
            .addServersItem(new Server()
                .url("http://localhost:8080")
                .description("Ambiente local"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")
                    .description("JWT de sessão (8h) ou token de integração")))
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"));
    }
}
```

### `application.yml` SpringDoc properties

```yaml
springdoc:
  swagger-ui:
    path: /swagger-ui.html
    operations-sorter: alpha
    tags-sorter: alpha
    display-request-duration: true
  api-docs:
    path: /api-docs
  show-actuator: false
```

---

## API Conventions

| Convention | Rule |
|---|---|
| **Base URL** | `/api/v1` |
| **Format** | `application/json` for all requests and responses |
| **Errors** | `application/problem+json` (RFC 7807 Problem Details) |
| **Auth** | `Authorization: Bearer <token>` |
| **Versioning** | URL path versioning (`/api/v1`, `/api/v2`) |
| **HTTP methods** | `GET` read, `POST` create, `PUT` full update, `PATCH` partial/state change, `DELETE` remove |
| **Status codes** | `200` OK, `201` Created, `204` No Content, `400` Validation, `401` Unauth, `404` Not Found, `422` Business rule, `423` Locked |
| **Dates** | ISO 8601 — `YYYY-MM-DD` for dates, `YYYY-MM-DDTHH:mm:ssZ` for timestamps |
| **Money** | Decimal with 2 places — `1500.00` (never cents) |
| **IDs** | `Long` integers in path params |
| **Pagination** | `page` (0-based), `size` (default 20, max 100), `sortBy`, `direction` (`ASC`/`DESC`) |
| **Pagination response** | `content`, `page`, `size`, `totalElements`, `totalPages`, `last` |

---

## Annotation Patterns

### Controller class

```java
@RestController
@RequestMapping("/api/v1/commitments")
@RequiredArgsConstructor
@Tag(name = "Commitments", description = "Notas de Empenho — emissão e gestão do ciclo de empenho")
public class CommitmentController { ... }
```

### Operation with responses

```java
@Operation(
    summary = "Issue a commitment (NE)",
    description = "Creates a new commitment against a budget allotment. " +
                  "Reserves the amount from the allotment's available balance atomically."
)
@ApiResponses({
    @ApiResponse(responseCode = "201", description = "Commitment issued",
        content = @Content(schema = @Schema(implementation = CommitmentResponse.class))),
    @ApiResponse(responseCode = "400", description = "Validation error",
        content = @Content(mediaType = "application/problem+json",
            schema = @Schema(implementation = ProblemDetail.class))),
    @ApiResponse(responseCode = "422", description = "Business rule violation " +
        "(insufficient balance, inactive vendor, or invalid allotment)",
        content = @Content(mediaType = "application/problem+json",
            schema = @Schema(implementation = ProblemDetail.class)))
})
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public CommitmentResponse issue(@RequestBody @Valid IssueCommitmentRequest request) { ... }
```

### Path parameter

```java
@Operation(summary = "Get commitment by ID")
@ApiResponse(responseCode = "404", description = "Commitment not found")
@GetMapping("/{id}")
public CommitmentResponse findById(
    @Parameter(description = "Commitment ID", example = "42")
    @PathVariable Long id) { ... }
```

### Query parameters on list endpoints

```java
@Operation(summary = "List commitments")
@GetMapping
public PageResponse<CommitmentResponse> list(
    @Parameter(description = "Filter by managing unit ID") @RequestParam(required = false) Long unitId,
    @Parameter(description = "Filter by fiscal year", example = "2025") @RequestParam(required = false) Integer fiscalYear,
    @Parameter(description = "Filter by status") @RequestParam(required = false) CommitmentStatus status,
    @Parameter(description = "Start of date range (YYYY-MM-DD)") @RequestParam(required = false) LocalDate startDate,
    @Parameter(description = "End of date range (YYYY-MM-DD)")   @RequestParam(required = false) LocalDate endDate,
    @Parameter(hidden = true) Pageable pageable) { ... }
```

### DTO (Java Record) with schema annotations

```java
@Schema(description = "Request body for issuing a new commitment")
public record IssueCommitmentRequest(
    @Schema(description = "Budget allotment ID", example = "10", requiredMode = REQUIRED)
    @NotNull Long allotmentId,

    @Schema(description = "Vendor ID", example = "5", requiredMode = REQUIRED)
    @NotNull Long vendorId,

    @Schema(description = "Managing unit ID", example = "1", requiredMode = REQUIRED)
    @NotNull Long managingUnitId,

    @Schema(description = "Commitment amount in BRL", example = "80000.00", requiredMode = REQUIRED)
    @NotNull @Positive BigDecimal value,

    @Schema(description = "Commitment type", allowableValues = {"ORDINARY", "ESTIMATED", "GLOBAL"}, requiredMode = REQUIRED)
    @NotNull CommitmentType commitmentType,

    @Schema(description = "Administrative process number", example = "23000.001234/2025-01")
    @NotBlank String processNumber,

    @Schema(description = "Object/purpose description", example = "Acquisition of IT equipment")
    @NotBlank String description,

    @Schema(description = "Fiscal year", example = "2025", requiredMode = REQUIRED)
    @NotNull @Min(2000) Integer fiscalYear
) {}
```

### Error response schema

```java
@Schema(description = "RFC 7807 Problem Details error response")
public record ProblemDetail(
    @Schema(example = "https://sifu.gov.br/errors/insufficient-balance") String type,
    @Schema(example = "Insufficient balance")                            String title,
    @Schema(example = "422")                                             int status,
    @Schema(example = "Available balance is R$ 1,000.00; requested: R$ 1,500.00") String detail,
    @Schema(example = "/api/v1/commitments")                            String instance
) {}
```

---

## HTTP Status Code Reference

| Status | Meaning | When |
|---|---|---|
| `200 OK` | Success | GET, PUT, PATCH responses |
| `201 Created` | Resource created | POST responses |
| `204 No Content` | Success, no body | DELETE, deactivation operations |
| `400 Bad Request` | Validation failure | `@Valid` fails, malformed JSON |
| `401 Unauthorized` | Not authenticated | Missing/invalid/expired token |
| `404 Not Found` | Resource missing | Entity not found |
| `422 Unprocessable Entity` | Business rule violation | Insufficient balance, invalid state transition |
| `423 Locked` | Account locked | Too many failed login attempts |

All non-2xx responses use RFC 7807 format with `Content-Type: application/problem+json`:

```json
{
  "type": "https://sifu.gov.br/errors/insufficient-balance",
  "title": "Insufficient balance",
  "status": 422,
  "detail": "Available balance for allotment 42 is R$ 1,000.00; requested: R$ 1,500.00",
  "instance": "/api/v1/commitments"
}
```

| Error type URI | HTTP | Exception |
|---|---|---|
| `.../errors/insufficient-balance` | 422 | `InsufficientBalanceException` |
| `.../errors/invalid-state-transition` | 422 | `InvalidStateTransitionException` |
| `.../errors/vendor-inactive` | 422 | `VendorInactiveException` |
| `.../errors/entity-not-found` | 404 | `EntityNotFoundException` |
| `.../errors/invalid-credentials` | 401 | `InvalidCredentialsException` |
| `.../errors/account-locked` | 423 | `AccountLockedException` |
| `.../errors/invalid-token` | 401 | `InvalidTokenException` |
| `.../errors/referenced-entity` | 422 | `ReferencedEntityException` |
| `.../errors/validation` | 400 | `MethodArgumentNotValidException` |

---

## Pagination

All list endpoints share a standard `Pageable` parameter block (handled transparently by Spring Data):

**Request parameters:**

| Parameter | Type | Default | Max | Description |
|---|---|---|---|---|
| `page` | int | 0 | — | 0-based page number |
| `size` | int | 20 | 100 | Items per page |
| `sort` | string | endpoint-specific | — | `fieldName,direction` e.g. `createdAt,DESC` |

**Response envelope (`PageResponse<T>`):**

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

---

## Endpoints

### Authentication — `@Tag(name = "Authentication")`

#### `POST /api/v1/auth/login` — public
```json
// Request
{ "login": "joao.silva", "password": "MinhaS3nha!" }

// 200 Response
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "expiresAt": "2025-05-21T18:00:00Z",
  "user": { "login": "joao.silva", "name": "João da Silva" }
}
// 401 — invalid credentials
// 423 — account locked (detail includes unlock time)
```

#### `POST /api/v1/auth/logout` — authenticated
Response `204 No Content`.

#### `POST /api/v1/auth/recover-password` — public
```json
{ "email": "joao@orgao.gov.br" }
// 204 always (never reveals if e-mail exists)
```

#### `PUT /api/v1/auth/reset-password` — public
```json
{ "resetToken": "abc-uuid-123", "newPassword": "NovaS3nha!" }
// 204 on success — 401 if token expired or already used
```

#### `PUT /api/v1/auth/change-password` — authenticated
```json
{ "currentPassword": "Atual123!", "newPassword": "Nova456!" }
// 204 on success — 401 if current password wrong
```

---

### Integration Tokens — `@Tag(name = "Integration Tokens")`

#### `GET /api/v1/tokens`
Lists tokens of the authenticated user. Token values are never returned after creation.

#### `POST /api/v1/tokens`
```json
// Request
{ "name": "Sistema de Compras", "expiresAt": "2026-12-31" }

// 201 Response — token value shown ONCE only
{ "id": 1, "name": "Sistema de Compras", "token": "sifu_tk_abc123...", "expiresAt": "2026-12-31T23:59:59Z" }
```

#### `DELETE /api/v1/tokens/{id}`
Revokes token immediately. Response `204`.

---

### Managing Units — `@Tag(name = "Managing Units")`

#### `GET /api/v1/managing-units`
Query params: `status` (`ACTIVE`/`INACTIVE`), `code`, `name`. Paginated.

#### `POST /api/v1/managing-units`
```json
// Request
{ "unitCode": "MIN_ED", "name": "Ministério da Educação", "parentUnitId": null, "description": "..." }
// 201 Response with generated id
```

#### `GET /api/v1/managing-units/{id}`

#### `PUT /api/v1/managing-units/{id}`
`unitCode` cannot be changed after creation — returns `422` if attempted.

#### `PATCH /api/v1/managing-units/{id}/deactivate`
Response `204`.

#### `GET /api/v1/managing-units/{id}/subordinates`
Returns recursive hierarchy of subordinate units.

---

### Budget Classifications — `@Tag(name = "Budget Classifications")`

Five resources share the same CRUD pattern:

| Path prefix | Entity |
|---|---|
| `/api/v1/budget-actions` | `BudgetAction` |
| `/api/v1/internal-plans` | `InternalPlan` |
| `/api/v1/expense-natures` | `ExpenseNature` |
| `/api/v1/funding-sources` | `FundingSource` |
| `/api/v1/ptres` | `Ptres` |

```
GET    /api/v1/{resource}             → paginated list
POST   /api/v1/{resource}             → create (201)
GET    /api/v1/{resource}/{id}        → detail (200)
PUT    /api/v1/{resource}/{id}        → full update (200)
PATCH  /api/v1/{resource}/{id}/deactivate → deactivate (204)
DELETE /api/v1/{resource}/{id}        → delete if unreferenced (204), 422 if referenced
```

---

### Budget Allotments — `@Tag(name = "Budget Allotments")`

#### `GET /api/v1/allotments`
Query params: `unitId`, `fiscalYear`, `budgetActionId`, `status`. Paginated.

#### `POST /api/v1/allotments`
```json
{
  "unitId": 1, "budgetActionId": 5, "internalPlanId": 3,
  "expenseNatureId": 2, "fundingSourceId": 1, "ptresId": 4,
  "initialValue": 500000.00, "fiscalYear": 2025
}
// 201 Response
```

#### `GET /api/v1/allotments/{id}`

#### `GET /api/v1/allotments/{id}/balance`
```json
{
  "allotmentId": 10,
  "fiscalYear": 2025,
  "initialAllotment": 500000.00,
  "updatedAllotment": 550000.00,
  "creditsReceived": 50000.00,
  "creditsTransferred": 0.00,
  "committed": 120000.00,
  "availableBalance": 430000.00
}
```

#### `POST /api/v1/allotments/{id}/supplement`
```json
// Request
{ "value": 100000.00 }
// 200 Response with updated allotment
```

---

### Credit Notes — `@Tag(name = "Credit Notes")`

#### `GET /api/v1/credit-notes`
Query params: `sourceUnitId`, `targetUnitId`, `status`, `fiscalYear`, `startDate`, `endDate`. Paginated.

#### `POST /api/v1/credit-notes`
```json
{
  "sourceUnitId": 1, "targetUnitId": 2,
  "sourceAllotmentId": 10, "targetAllotmentId": 11,
  "value": 50000.00, "issueDate": "2025-05-20",
  "description": "Suplementação para custeio"
}
// 201 Response — includes generated creditNoteNumber
// 422 — sourceUnitId == targetUnitId
```

#### `GET /api/v1/credit-notes/{id}`

#### `PATCH /api/v1/credit-notes/{id}/approve`
Validates available balance of source allotment. Response `200` with updated credit note.
- `422` — insufficient balance or status ≠ `PENDING`

#### `PATCH /api/v1/credit-notes/{id}/cancel`
Only if `status = PENDING`. Response `200`.
- `422` — status ≠ `PENDING`

#### `PATCH /api/v1/credit-notes/{id}/reverse`
Only if `status = APPROVED`. Validates no active commitments depend on target allotment credits.
Response `200`.
- `422` — active commitments exist or status ≠ `APPROVED`

---

### Vendors — `@Tag(name = "Vendors")`

#### `GET /api/v1/vendors`
Query params: `cnpj`, `name`, `status`, `personType`. Paginated.

#### `POST /api/v1/vendors`
```json
{
  "cnpj": "12345678000190", "name": "Empresa ABC Ltda",
  "personType": "LEGAL_ENTITY",
  "bank": "001", "branch": "1234", "accountNumber": "56789-0"
}
// 201 Response
// 422 — duplicate CNPJ
```

#### `GET /api/v1/vendors/{id}`

#### `PUT /api/v1/vendors/{id}`

#### `PATCH /api/v1/vendors/{id}/deactivate`
Response `204`.

---

### Commitments — `@Tag(name = "Commitments")`

#### `GET /api/v1/commitments`
Query params: `unitId`, `allotmentId`, `vendorId`, `commitmentType`, `status`, `startDate`, `endDate`. Paginated.

#### `POST /api/v1/commitments`
```json
{
  "unitId": 1, "allotmentId": 10, "vendorId": 5,
  "value": 80000.00, "issueDate": "2025-05-20",
  "commitmentType": "ORDINARY",
  "processNumber": "23000.001234/2025-01",
  "description": "Acquisition of IT equipment",
  "fiscalYear": 2025
}
// 201 Response — includes generated commitmentNumber
// 422 — insufficient balance, inactive vendor
```

#### `GET /api/v1/commitments/{id}`

#### `POST /api/v1/commitments/{id}/reinforce`
Increases commitment value. Only if `status ≠ VOIDED`.
```json
{ "value": 20000.00 }
// 200 Response with updated commitment
// 422 — insufficient allotment balance or invalid status
```

#### `PATCH /api/v1/commitments/{id}/void`
```json
// ORDINARY: full void only
{ "voidType": "TOTAL" }

// ESTIMATED / GLOBAL: partial void allowed
{ "voidType": "PARTIAL", "value": 10000.00 }

// 422 — has active settlements, or ORDINARY with PARTIAL
```

#### `GET /api/v1/commitments/{id}/settlements`
Lists settlements linked to this commitment. Paginated.

---

### Settlements — `@Tag(name = "Settlements")`

#### `GET /api/v1/settlements`
Query params: `commitmentId`, `status`, `startDate`, `endDate`. Paginated.

#### `POST /api/v1/settlements`
```json
{
  "commitmentId": 7, "value": 80000.00,
  "settlementDate": "2025-05-20",
  "fiscalDocument": "NF-123456"
}
// 201 Response — includes generated settlementNumber
// 422 — commitment VOIDED or PAID, or value exceeds remaining balance
// 422 — ORDINARY commitment: partial settlement not allowed
```

#### `GET /api/v1/settlements/{id}`

#### `PATCH /api/v1/settlements/{id}/reverse`
Only if `status = REGISTERED` and no active payment order linked.
Response `200`.
- `422` — linked payment order exists or status ≠ `REGISTERED`

---

### Payment Orders — `@Tag(name = "Payment Orders")`

#### `GET /api/v1/payment-orders`
Query params: `settlementId`, `status`, `startDate`, `endDate`. Paginated.

#### `POST /api/v1/payment-orders`
```json
{
  "settlementId": 3, "value": 80000.00,
  "bank": "001", "branch": "1234", "destinationAccount": "56789-0"
}
// 201 Response — includes generated paymentOrderNumber
// 422 — settlement status ≠ REGISTERED, or duplicate active payment order
```

#### `GET /api/v1/payment-orders/{id}`

#### `PATCH /api/v1/payment-orders/{id}/process`
Marks as `PROCESSED` (simulates bank confirmation). Updates linked commitment to `PAID`.
Response `200`.
- `422` — status ≠ `ISSUED`

#### `PATCH /api/v1/payment-orders/{id}/cancel`
Only if `status = ISSUED`. Response `200`.
- `422` — status ≠ `ISSUED`

---

### Reports — `@Tag(name = "Reports")`

#### `GET /api/v1/reports/budget-execution`
Required params: `unitId`, `fiscalYear`.
Optional: `budgetActionId`, `internalPlanId`, `expenseNatureId`, `fundingSourceId`, `ptresId`, `startDate`, `endDate`.

```json
{
  "filters": { "unitId": 1, "fiscalYear": 2025 },
  "totals": {
    "initialAllotment":  2000000.00,
    "updatedAllotment":  2200000.00,
    "committed":          800000.00,
    "pendingSettlement":  200000.00,
    "settled":            600000.00,
    "pendingPayment":     100000.00,
    "paid":               500000.00,
    "availableBalance":  1400000.00
  },
  "byAllotment": [
    {
      "allotmentId": 10,
      "budgetAction": "2109 — Manutenção Administrativa",
      "committed": 80000.00,
      "settled": 80000.00,
      "paid": 80000.00,
      "availableBalance": 420000.00
    }
  ]
}
```

#### `GET /api/v1/reports/dashboard`
Params: `unitId`, `fiscalYear`.

```json
{
  "unitId": 1, "fiscalYear": 2025,
  "availableCredit":      1400000.00,
  "totalCommitted":        800000.00,
  "totalSettled":          600000.00,
  "totalPaid":             500000.00,
  "executionPercentage":      36.36
}
```

---

### Audit Log — `@Tag(name = "Audit")`

#### `GET /api/v1/audit-log`
Query params: `userLogin`, `entity`, `operation`, `startDate`, `endDate`, `unitId`. Paginated.

```json
// Single audit entry
{
  "id": 1001,
  "userLogin": "joao.silva",
  "operation": "ISSUE_COMMITMENT",
  "entity": "Commitment",
  "entityId": 42,
  "ipAddress": "192.168.1.10",
  "before": null,
  "after": { "id": 42, "value": 80000.00, "status": "PENDING_SETTLEMENT" },
  "occurredAt": "2025-05-20T14:32:11Z"
}
```

---

### Users — `@Tag(name = "Users")`

```
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/{id}
PUT    /api/v1/users/{id}
PATCH  /api/v1/users/{id}/deactivate
```

---

## Security — Bearer JWT

All authenticated endpoints require:

```
Authorization: Bearer <token>
```

The Swagger UI includes a **"Authorize" button** (configured via the `SecurityScheme` bean) that injects this header into all test requests. Users can paste a JWT obtained from `POST /api/v1/auth/login` directly into the UI.

Two token types are accepted:
- **Session JWT** — issued by login, expires in 8h, claim `type=SESSION`
- **Integration token** — issued via `/api/v1/tokens`, configurable expiry, claim `type=INTEGRATION`

---

## API Versioning

Current version: **v1** (path prefix `/api/v1`).

Future version strategy:
- New major version = new prefix `/api/v2`
- v1 controllers remain unchanged — no breaking changes within a version
- Deprecation communicated via `@Deprecated` on the controller + `Deprecation` response header
