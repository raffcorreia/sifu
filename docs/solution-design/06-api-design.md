# 06 — Design da API

> **Camada:** Core Design | **Depende de:** 05-backend-design

## Convenções

- Base URL: `/api/v1`
- Formato: JSON (`Content-Type: application/json`)
- Autenticação: `Authorization: Bearer <token>`
- Paginação: parâmetros `page`, `size`, `sortBy`, `direction`
- Erros: RFC 7807 (Problem Details)
- Datas: ISO 8601 (`YYYY-MM-DD` para datas, `YYYY-MM-DDTHH:mm:ssZ` para timestamps)
- Valores monetários: número decimal com 2 casas (`1500.00`)

---

## Autenticação

### POST `/api/v1/auth/login`
```json
// Requisição
{ "login": "joao.silva", "password": "MinhaS3nha!" }

// Resposta 200
{ "token": "eyJ...", "expiresAt": "2025-05-21T10:00:00Z", "user": { "login": "joao.silva", "name": "João Silva" } }

// Erro 401 — credenciais inválidas
// Erro 423 — conta bloqueada
```

### POST `/api/v1/auth/logout`
Invalida a sessão atual. Requer autenticação.

### POST `/api/v1/auth/recover-password`
```json
{ "email": "joao@orgao.gov.br" }
// Resposta 204 (sempre, para não expor se e-mail existe)
```

### PUT `/api/v1/auth/reset-password`
```json
{ "resetToken": "abc123", "newPassword": "NovaS3nha!" }
```

### PUT `/api/v1/auth/change-password`
```json
{ "currentPassword": "Atual123!", "newPassword": "Nova456!" }
```

---

## Tokens de Integração

### GET `/api/v1/tokens`
Lista tokens do usuário autenticado (metadados; sem o valor do token).

### POST `/api/v1/tokens`
```json
// Requisição
{ "name": "Sistema de Compras", "expiresAt": "2026-12-31" }

// Resposta 201 — token exibido uma única vez
{ "id": 1, "name": "Sistema de Compras", "token": "sifu_tk_abc123...", "expiresAt": "2026-12-31T23:59:59Z" }
```

### DELETE `/api/v1/tokens/{id}`
Revoga o token. Resposta 204.

---

## Unidades Gestoras

### GET `/api/v1/managing-units`
Filtros: `status`, `code`, `name`. Paginado.

### POST `/api/v1/managing-units`
```json
{ "unitCode": "MIN_ED", "name": "Ministério da Educação", "parentUnitId": null, "description": "..." }
// Resposta 201
```

### GET `/api/v1/managing-units/{id}`

### PUT `/api/v1/managing-units/{id}`

### PATCH `/api/v1/managing-units/{id}/deactivate`
Resposta 204.

### GET `/api/v1/managing-units/{id}/subordinates`
Retorna hierarquia de subordinadas (recursiva).

---

## Classificações Orçamentárias

Padrão idêntico para: `/api/v1/budget-actions`, `/api/v1/internal-plans`, `/api/v1/expense-natures`, `/api/v1/funding-sources`, `/api/v1/ptres`.

```
GET    /api/v1/{recurso}        → lista paginada
POST   /api/v1/{recurso}        → cria (201)
GET    /api/v1/{recurso}/{id}   → detalhe
PUT    /api/v1/{recurso}/{id}   → atualiza
PATCH  /api/v1/{recurso}/{id}/deactivate → desativa (204)
DELETE /api/v1/{recurso}/{id}   → exclui (somente se nunca referenciado; 204)
```

---

## Dotações Orçamentárias

### GET `/api/v1/allotments`
Filtros: `unitId`, `fiscalYear`, `budgetActionId`, `status`.

### POST `/api/v1/allotments`
```json
{
  "unitId": 1, "budgetActionId": 5, "internalPlanId": 3,
  "expenseNatureId": 2, "fundingSourceId": 1, "ptresId": 4,
  "initialValue": 500000.00, "fiscalYear": 2025
}
// Resposta 201
```

### GET `/api/v1/allotments/{id}/saldo`
```json
{
  "initialAllotment": 500000.00,
  "updatedAllotment": 550000.00,
  "creditsReceived": 50000.00,
  "creditsTransferred": 0.00,
  "committed": 120000.00,
  "availableBalance": 430000.00
}
```

### POST `/api/v1/allotments/{id}/supplement`
```json
{ "value": 100000.00 }
// Resposta 200 com dotação atualizada
```

---

## Notas de Crédito

### GET `/api/v1/credit-notes`
Filtros: `sourceUnitId`, `targetUnitId`, `status`, `fiscalYear`, `dataInicio`, `dataFim`.

### POST `/api/v1/credit-notes`
```json
{
  "sourceUnitId": 1, "targetUnitId": 2,
  "sourceAllotmentId": 10, "targetAllotmentId": 11,
  "value": 50000.00, "issueDate": "2025-05-20"
}
// Resposta 201 com numero_nc gerado
```

### GET `/api/v1/credit-notes/{id}`

### PATCH `/api/v1/credit-notes/{id}/approve`
Resposta 200 com NC atualizada.

### PATCH `/api/v1/credit-notes/{id}/cancel`
Resposta 200 com NC atualizada.

### PATCH `/api/v1/credit-notes/{id}/reverse`
Resposta 200 com NC atualizada.

---

## Fornecedores

### GET `/api/v1/vendors`
Filtros: `cnpj`, `name`, `status`.

### POST `/api/v1/vendors`
```json
{
  "cnpj": "12345678000190", "name": "Empresa ABC Ltda",
  "personType": "JURIDICA", "bank": "001", "branch": "1234", "accountNumber": "56789-0"
}
```

### GET, PUT `/api/v1/vendors/{id}`

### PATCH `/api/v1/vendors/{id}/deactivate`

---

## Notas de Empenho

### GET `/api/v1/commitments`
Filtros: `unitId`, `allotmentId`, `vendorId`, `commitmentType`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/commitments`
```json
{
  "unitId": 1, "allotmentId": 10, "vendorId": 5,
  "amount": 80000.00, "issueDate": "2025-05-20",
  "commitmentType": "ORDINARIO",
  "processNumber": "23000.001234/2025-01",
  "objectDescription": "Aquisição de equipamentos de informática"
}
// Resposta 201 com numero_empenho gerado
```

### GET `/api/v1/commitments/{id}`

### POST `/api/v1/commitments/{id}/reinforce`
```json
{ "value": 20000.00 }
// Resposta 200 com NE atualizada
```

### PATCH `/api/v1/commitments/{id}/void`
```json
{ "tipo": "TOTAL" }
// ou
{ "tipo": "PARCIAL", "value": 10000.00 }
```

### GET `/api/v1/commitments/{id}/settlements`
Lista liquidações vinculadas à NE.

---

## Liquidações de Empenho

### GET `/api/v1/settlements`
Filtros: `commitmentId`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/settlements`
```json
{
  "commitmentId": 7, "amount": 80000.00,
  "settlementDate": "2025-05-20", "fiscalDocument": "NF-123456"
}
// Resposta 201 com numero_liquidacao gerado
```

### GET `/api/v1/settlements/{id}`

### PATCH `/api/v1/settlements/{id}/reverse`
Resposta 200 com NL atualizada.

---

## Ordens Bancárias

### GET `/api/v1/payment-orders`
Filtros: `settlementId`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/payment-orders`
```json
{
  "settlementId": 3, "amount": 80000.00,
  "bank": "001", "branch": "1234", "destinationAccount": "56789-0"
}
// Resposta 201 com numero_ob gerado
```

### GET `/api/v1/payment-orders/{id}`

### PATCH `/api/v1/payment-orders/{id}/process`
Marca como processada (simulação de confirmação bancária).

### PATCH `/api/v1/payment-orders/{id}/cancel`

---

## Consultas

### GET `/api/v1/reports/budget-execution`
Filtros: `unitId` (obrigatório), `fiscalYear` (obrigatório), `budgetActionId`, `internalPlanId`, `expenseNatureId`, `fundingSourceId`, `ptresId`, `dataInicio`, `dataFim`.

```json
// Resposta
{
  "filters": { "unitId": 1, "fiscalYear": 2025 },
  "totals": {
    "initialAllotment": 2000000.00, "updatedAllotment": 2200000.00,
    "committed": 800000.00, "pendingSettlement": 200000.00,
    "settled": 600000.00, "pendingPayment": 100000.00,
    "paid": 500000.00, "availableBalance": 1400000.00
  },
  "byAllotment": [...]
}
```

### GET `/api/v1/reports/dashboard`
Parâmetros: `unitId`, `fiscalYear`.

```json
{
  "availableCredit": 1400000.00,
  "totalCommitted": 800000.00,
  "totalSettled": 600000.00,
  "totalPaid": 500000.00,
  "executionPercentage": 36.36
}
```

---

## Auditoria

### GET `/api/v1/audit-log`
Filtros: `userLogin`, `entidade`, `operacao`, `dataInicio`, `dataFim`, `unitId`. Paginado.

---

## Usuários (Administração)

```
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/{id}
PUT    /api/v1/users/{id}
PATCH  /api/v1/users/{id}/deactivate
```

---

## Documentação Swagger

Disponível em: `GET /swagger-ui.html` e `GET /api-docs`

Todos os endpoints possuem:
- Descrição da operação
- Parâmetros documentados
- Exemplos de requisição e resposta
- Códigos de erro possíveis
