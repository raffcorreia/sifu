# 06 — Design da API

> **Camada:** Core Design | **Depende de:** 05-backend-design

## Convenções

- Base URL: `/api/v1`
- Formato: JSON (`Content-Type: application/json`)
- Autenticação: `Authorization: Bearer <token>`
- Paginação: parâmetros `pagina`, `tamanho`, `ordenarPor`, `direcao`
- Erros: RFC 7807 (Problem Details)
- Datas: ISO 8601 (`YYYY-MM-DD` para datas, `YYYY-MM-DDTHH:mm:ssZ` para timestamps)
- Valores monetários: número decimal com 2 casas (`1500.00`)

---

## Autenticação

### POST `/api/v1/auth/login`
```json
// Requisição
{ "login": "joao.silva", "senha": "MinhaS3nha!" }

// Resposta 200
{ "token": "eyJ...", "expiracao": "2025-05-21T10:00:00Z", "usuario": { "login": "joao.silva", "nome": "João Silva" } }

// Erro 401 — credenciais inválidas
// Erro 423 — conta bloqueada
```

### POST `/api/v1/auth/logout`
Invalida a sessão atual. Requer autenticação.

### POST `/api/v1/auth/recuperar-senha`
```json
{ "email": "joao@orgao.gov.br" }
// Resposta 204 (sempre, para não expor se e-mail existe)
```

### PUT `/api/v1/auth/redefinir-senha`
```json
{ "tokenRedefinicao": "abc123", "novaSenha": "NovaS3nha!" }
```

### PUT `/api/v1/auth/alterar-senha`
```json
{ "senhaAtual": "Atual123!", "novaSenha": "Nova456!" }
```

---

## Tokens de Integração

### GET `/api/v1/tokens`
Lista tokens do usuário autenticado (metadados; sem o valor do token).

### POST `/api/v1/tokens`
```json
// Requisição
{ "nome": "Sistema de Compras", "expiracao": "2026-12-31" }

// Resposta 201 — token exibido uma única vez
{ "id": 1, "nome": "Sistema de Compras", "token": "sifu_tk_abc123...", "expiracao": "2026-12-31T23:59:59Z" }
```

### DELETE `/api/v1/tokens/{id}`
Revoga o token. Resposta 204.

---

## Unidades Gestoras

### GET `/api/v1/unidades-gestoras`
Filtros: `status`, `codigo`, `nome`. Paginado.

### POST `/api/v1/unidades-gestoras`
```json
{ "codigoUg": "MIN_ED", "nome": "Ministério da Educação", "orgaoSuperiorId": null, "descricao": "..." }
// Resposta 201
```

### GET `/api/v1/unidades-gestoras/{id}`

### PUT `/api/v1/unidades-gestoras/{id}`

### PATCH `/api/v1/unidades-gestoras/{id}/desativar`
Resposta 204.

### GET `/api/v1/unidades-gestoras/{id}/subordinadas`
Retorna hierarquia de subordinadas (recursiva).

---

## Classificações Orçamentárias

Padrão idêntico para: `/api/v1/acoes-orcamentarias`, `/api/v1/planos-internos`, `/api/v1/naturezas-despesa`, `/api/v1/fontes-recurso`, `/api/v1/ptres`.

```
GET    /api/v1/{recurso}        → lista paginada
POST   /api/v1/{recurso}        → cria (201)
GET    /api/v1/{recurso}/{id}   → detalhe
PUT    /api/v1/{recurso}/{id}   → atualiza
PATCH  /api/v1/{recurso}/{id}/desativar → desativa (204)
DELETE /api/v1/{recurso}/{id}   → exclui (somente se nunca referenciado; 204)
```

---

## Dotações Orçamentárias

### GET `/api/v1/dotacoes`
Filtros: `ugId`, `exercicio`, `acaoOrcamentariaId`, `status`.

### POST `/api/v1/dotacoes`
```json
{
  "ugId": 1, "acaoOrcamentariaId": 5, "planoInternoId": 3,
  "naturezaDespesaId": 2, "fonteRecursoId": 1, "ptresId": 4,
  "valorInicial": 500000.00, "exercicio": 2025
}
// Resposta 201
```

### GET `/api/v1/dotacoes/{id}/saldo`
```json
{
  "dotacaoInicial": 500000.00,
  "dotacaoAtualizada": 550000.00,
  "creditosRecebidos": 50000.00,
  "creditosCedidos": 0.00,
  "empenhado": 120000.00,
  "saldoDisponivel": 430000.00
}
```

### POST `/api/v1/dotacoes/{id}/suplementar`
```json
{ "valor": 100000.00 }
// Resposta 200 com dotação atualizada
```

---

## Notas de Crédito

### GET `/api/v1/notas-credito`
Filtros: `ugOrigemId`, `ugDestinoId`, `status`, `exercicio`, `dataInicio`, `dataFim`.

### POST `/api/v1/notas-credito`
```json
{
  "ugOrigemId": 1, "ugDestinoId": 2,
  "dotacaoOrigemId": 10, "dotacaoDestinoId": 11,
  "valor": 50000.00, "dataEmissao": "2025-05-20"
}
// Resposta 201 com numero_nc gerado
```

### GET `/api/v1/notas-credito/{id}`

### PATCH `/api/v1/notas-credito/{id}/aprovar`
Resposta 200 com NC atualizada.

### PATCH `/api/v1/notas-credito/{id}/cancelar`
Resposta 200 com NC atualizada.

### PATCH `/api/v1/notas-credito/{id}/estornar`
Resposta 200 com NC atualizada.

---

## Fornecedores

### GET `/api/v1/fornecedores`
Filtros: `cnpj`, `nome`, `status`.

### POST `/api/v1/fornecedores`
```json
{
  "cnpj": "12345678000190", "nome": "Empresa ABC Ltda",
  "tipoPessoa": "JURIDICA", "banco": "001", "agencia": "1234", "contaCorrente": "56789-0"
}
```

### GET, PUT `/api/v1/fornecedores/{id}`

### PATCH `/api/v1/fornecedores/{id}/desativar`

---

## Notas de Empenho

### GET `/api/v1/notas-empenho`
Filtros: `ugId`, `dotacaoId`, `fornecedorId`, `tipoEmpenho`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/notas-empenho`
```json
{
  "ugId": 1, "dotacaoId": 10, "fornecedorId": 5,
  "valor": 80000.00, "dataEmissao": "2025-05-20",
  "tipoEmpenho": "ORDINARIO",
  "processo": "23000.001234/2025-01",
  "descricaoObjeto": "Aquisição de equipamentos de informática"
}
// Resposta 201 com numero_empenho gerado
```

### GET `/api/v1/notas-empenho/{id}`

### POST `/api/v1/notas-empenho/{id}/reforco`
```json
{ "valor": 20000.00 }
// Resposta 200 com NE atualizada
```

### PATCH `/api/v1/notas-empenho/{id}/anular`
```json
{ "tipo": "TOTAL" }
// ou
{ "tipo": "PARCIAL", "valor": 10000.00 }
```

### GET `/api/v1/notas-empenho/{id}/liquidacoes`
Lista liquidações vinculadas à NE.

---

## Liquidações de Empenho

### GET `/api/v1/liquidacoes`
Filtros: `notaEmpenhoId`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/liquidacoes`
```json
{
  "notaEmpenhoId": 7, "valor": 80000.00,
  "dataLiquidacao": "2025-05-20", "documentoFiscal": "NF-123456"
}
// Resposta 201 com numero_liquidacao gerado
```

### GET `/api/v1/liquidacoes/{id}`

### PATCH `/api/v1/liquidacoes/{id}/estornar`
Resposta 200 com NL atualizada.

---

## Ordens Bancárias

### GET `/api/v1/ordens-bancarias`
Filtros: `liquidacaoId`, `status`, `dataInicio`, `dataFim`.

### POST `/api/v1/ordens-bancarias`
```json
{
  "liquidacaoId": 3, "valor": 80000.00,
  "banco": "001", "agencia": "1234", "contaDestino": "56789-0"
}
// Resposta 201 com numero_ob gerado
```

### GET `/api/v1/ordens-bancarias/{id}`

### PATCH `/api/v1/ordens-bancarias/{id}/processar`
Marca como processada (simulação de confirmação bancária).

### PATCH `/api/v1/ordens-bancarias/{id}/cancelar`

---

## Consultas

### GET `/api/v1/consultas/execucao-orcamentaria`
Filtros: `ugId` (obrigatório), `exercicio` (obrigatório), `acaoOrcamentariaId`, `planoInternoId`, `naturezaDespesaId`, `fonteRecursoId`, `ptresId`, `dataInicio`, `dataFim`.

```json
// Resposta
{
  "filtros": { "ugId": 1, "exercicio": 2025 },
  "totais": {
    "dotacaoInicial": 2000000.00, "dotacaoAtualizada": 2200000.00,
    "empenhado": 800000.00, "aLiquidar": 200000.00,
    "liquidado": 600000.00, "aPagar": 100000.00,
    "pago": 500000.00, "saldoDisponivel": 1400000.00
  },
  "porDotacao": [...]
}
```

### GET `/api/v1/consultas/dashboard`
Parâmetros: `ugId`, `exercicio`.

```json
{
  "creditoDisponivel": 1400000.00,
  "totalEmpenhado": 800000.00,
  "totalLiquidado": 600000.00,
  "totalPago": 500000.00,
  "percentualExecutado": 36.36
}
```

---

## Auditoria

### GET `/api/v1/auditoria`
Filtros: `usuarioLogin`, `entidade`, `operacao`, `dataInicio`, `dataFim`, `ugId`. Paginado.

---

## Usuários (Administração)

```
GET    /api/v1/usuarios
POST   /api/v1/usuarios
GET    /api/v1/usuarios/{id}
PUT    /api/v1/usuarios/{id}
PATCH  /api/v1/usuarios/{id}/desativar
```

---

## Documentação Swagger

Disponível em: `GET /swagger-ui.html` e `GET /api-docs`

Todos os endpoints possuem:
- Descrição da operação
- Parâmetros documentados
- Exemplos de requisição e resposta
- Códigos de erro possíveis
