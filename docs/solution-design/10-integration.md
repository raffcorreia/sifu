# 10 — Integração com Sistemas Externos

> **Camada:** Operations | **Depende de:** 06-api-design, 08-security-design

## Visão Geral

O SIFU expõe uma API REST versionada para que sistemas externos possam integrar-se sem acesso à interface web. A autenticação é feita exclusivamente via **token de integração** gerado por um usuário autenticado.

## Como Obter um Token de Integração

1. Um usuário administrador faz login na interface web.
2. Navega para Administração → Tokens de Integração → Novo Token.
3. Define um nome descritivo (ex.: "Sistema de Compras") e data de expiração.
4. O token é exibido **uma única vez** — deve ser copiado e armazenado com segurança.
5. O sistema externo usa o token em todas as requisições.

## Autenticação nas Requisições

```http
GET /api/v1/commitments
Authorization: Bearer sifu_tk_abc123...
Content-Type: application/json
```

## Exemplos de Uso

### Consultar saldo de uma dotação

```http
GET /api/v1/allotments/42/balance
Authorization: Bearer sifu_tk_abc123...
```

```json
// Resposta 200
{
  "initialAllotment": 500000.00,
  "updatedAllotment": 550000.00,
  "creditReceived": 50000.00,
  "creditGiven": 0.00,
  "committed": 120000.00,
  "availableBalance": 430000.00
}
```

### Emitir uma Nota de Empenho

```http
POST /api/v1/commitments
Authorization: Bearer sifu_tk_abc123...
Content-Type: application/json

{
  "managingUnitId": 1,
  "allotmentId": 42,
  "vendorId": 7,
  "amount": 80000.00,
  "issueDate": "2025-05-20",
  "commitmentType": "ORDINARY",
  "process": "23000.001234/2025-01",
  "objectDescription": "Aquisição de equipamentos de TI"
}
```

```json
// Resposta 201
{
  "id": 103,
  "commitmentNumber": "2025MIN_ED000103",
  "status": "PENDING_SETTLEMENT",
  "amount": 80000.00,
  ...
}
```

### Consultar execução orçamentária

```http
GET /api/v1/queries/budget-execution?managingUnitId=1&fiscalYear=2025
Authorization: Bearer sifu_tk_abc123...
```

## Rate Limiting

| Limite | Valor |
|---|---|
| Requisições por minuto por token | 100 |
| Resposta ao exceder | 429 Too Many Requests |
| Header de resposta | `Retry-After: 30` (segundos) |

## Erros Comuns

| Código | Causa | Solução |
|---|---|---|
| 401 | Token ausente ou inválido | Verificar header Authorization |
| 401 | Token expirado | Gerar novo token na interface web |
| 401 | Token revogado | Verificar com o administrador |
| 422 | Saldo insuficiente | Consultar saldo antes de emitir |
| 422 | Transição de estado inválida | Verificar status atual do documento |
| 429 | Rate limit excedido | Aguardar e reduzir frequência de chamadas |

## Documentação Interativa

Disponível em: `https://sifu/swagger-ui.html`

A documentação inclui todos os endpoints, parâmetros, exemplos de payload e permite testar diretamente com o token de integração.

## Integrações Previstas para Fase 2

| Sistema | Tipo | Direção | Uso |
|---|---|---|---|
| Sistema de Compras | REST | Entrada | Geração automática de NE após contrato |
| Sistema Bancário | REST | Saída | Confirmação de processamento de OB |
| Sistema de Contratos | REST | Entrada | Vinculação de contrato a empenho |
