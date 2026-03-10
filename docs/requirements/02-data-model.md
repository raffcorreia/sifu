# 02 — Modelo de Dados

## Organização de Múltiplos Órgãos

O SIFU suporta múltiplos órgãos e instituições através de uma **hierarquia de Unidades Gestoras (UG)**:

- Cada órgão é representado como uma UG na raiz da hierarquia
- Subórgãos ou departamentos podem ser representados como UGs subordinadas (através do campo `orgao_superior`)
- Notas de Crédito (NC) permitem transferência de recursos entre órgãos diferentes
- Cada UG possui suas próprias dotações orçamentárias, empenhos e operações

**Exemplo de hierarquia:**
```
Ministério X
├── Órgão A
│   ├── Departamento A1
│   └── Departamento A2
├── Órgão B
│   └── Departamento B1
└── Órgão C
```

## Entidades Principais

- UnidadeGestora (UG)
- Gestao
- AcaoOrcamentaria
- PlanoInterno (PI)
- NaturezaDespesa (ND)
- FonteRecurso
- PTRES
- DotacaoOrcamentaria
- CreditoOrcamentario
- NotaCredito (NC)
- Fornecedor
- NotaEmpenho (NE)
- LiquidacaoEmpenho (NL)
- OrdemBancaria (OB)
- Usuario
- PerfilAcesso

## Relacionamentos entre Entidades

- UG 1 --- N DotacaoOrcamentaria
- DotacaoOrcamentaria
  - → AcaoOrcamentaria
  - → PlanoInterno
  - → NaturezaDespesa
  - → FonteRecurso
  - → PTRES
- NotaCredito
  - → UG Origem
  - → UG Destino
  - → DotacaoOrcamentaria
- NotaEmpenho
  - → DotacaoOrcamentaria
  - → Fornecedor
- LiquidacaoEmpenho
  - → NotaEmpenho
- OrdemBancaria
  - → LiquidacaoEmpenho

## CRUDs Necessários

### 4.1 Unidade Gestora (UG)

Representa órgãos, instituições, departamentos na hierarquia do sistema.

**Campos:**

- id
- codigo_ug (código único que identifica a UG no órgão superior)
- nome
- orgao_superior (referência para hierarquia)
- status (ativo/inativo)
- descricao (descrição da unidade gestora)

**Operações:**

- CREATE UG
- READ UG
- UPDATE UG
- LIST UG
- LIST UGs subordinadas

**Exemplo:**
- UG: Ministério da Educação (código: MIN_ED, orgao_superior: NULL)
- UG: Secretaria de Ensino Superior (código: SES, orgao_superior: MIN_ED)
- UG: Instituto X (código: INST_X, orgao_superior: SES)

### 4.2 Classificações Orçamentárias

#### Ação Orçamentária

**Campos:**

- id
- codigo
- descricao
- exercicio

CRUD completo.

#### Plano Interno (PI)

**Campos:**

- id
- codigo
- descricao
- acao_orcamentaria_id

CRUD completo.

#### Natureza da Despesa (ND)

**Campos:**

- id
- codigo
- descricao
- tipo

CRUD completo.

#### Fonte de Recurso

**Campos:**

- id
- codigo
- descricao

CRUD completo.

#### PTRES

**Campos:**

- id
- codigo
- descricao

CRUD completo.

## 5. Dotação Orçamentária

Representa o orçamento disponível.

**Campos:**

- id
- ug_id
- acao_orcamentaria_id
- plano_interno_id
- natureza_despesa_id
- fonte_recurso_id
- ptres_id
- valor_inicial
- valor_atualizado
- exercicio

**Operações:**

- CREATE dotação
- READ dotação
- UPDATE dotação
- CONSULT saldo

**Saldo calculado:**

```
saldo = dotacao + creditos_recebidos - empenhos_emitidos
```

## 6. Nota de Crédito (NC)

Usada para **transferência de crédito orçamentário**.

**Campos:**

- id
- numero_nc
- ug_origem
- ug_destino
- dotacao_origem
- dotacao_destino
- valor
- data_emissao
- status

**Operações:**

- CRIAR NC
- APROVAR NC
- CANCELAR NC
- CONSULTAR NC

**Regras:**

- valor NC <= saldo disponível

## 7. Fornecedor

**Campos:**

- id
- cnpj
- nome
- tipo_pessoa
- status

**Operações:**

- CREATE fornecedor
- READ fornecedor
- UPDATE fornecedor
- SEARCH fornecedor

## 8. Nota de Empenho (NE)

Reserva de orçamento para despesa.

**Campos:**

- id
- numero_empenho
- ug_id
- dotacao_id
- fornecedor_id
- natureza_despesa
- plano_interno
- valor
- data_emissao
- tipo_empenho
- status

**Tipos:**

- ORDINARIO
- ESTIMATIVO
- GLOBAL

**Operações:**

- CRIAR empenho
- ANULAR empenho
- CONSULTAR empenho
- LISTAR empenhos

**Regras:**

- valor empenho <= saldo da dotação

## 9. Liquidação de Empenho (NL)

Confirma que o serviço foi prestado.

**Campos:**

- id
- numero_liquidacao
- nota_empenho_id
- valor
- data_liquidacao
- documento_fiscal
- status

**Operações:**

- CRIAR liquidação
- CONSULTAR liquidação

## 10. Ordem Bancária (OB)

Pagamento ao fornecedor.

**Campos:**

- id
- numero_ob
- liquidacao_id
- valor
- data_pagamento
- banco
- status

**Operações:**

- CRIAR ordem bancária
- CONSULTAR pagamento