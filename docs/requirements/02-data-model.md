# 02 — Modelo de Dados

## Organização de Múltiplos Órgãos

O SIFU suporta múltiplos órgãos e instituições através de uma hierarquia de **Unidades Gestoras (UG)**:

- Cada órgão é representado como uma UG na raiz da hierarquia
- Subórgãos ou departamentos são representados como UGs subordinadas (campo `orgaoSuperior`)
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
- NotaCredito (NC)
- Fornecedor
- NotaEmpenho (NE)
- LiquidacaoEmpenho (NL)
- OrdemBancaria (OB)
- Usuario

## Relacionamentos entre Entidades

```
UG 1 --- N DotacaoOrcamentaria

DotacaoOrcamentaria
  → AcaoOrcamentaria
  → PlanoInterno
  → NaturezaDespesa
  → FonteRecurso
  → PTRES

NotaCredito
  → UG Origem
  → UG Destino
  → DotacaoOrcamentaria Origem
  → DotacaoOrcamentaria Destino

NotaEmpenho
  → DotacaoOrcamentaria
  → Fornecedor

LiquidacaoEmpenho
  → NotaEmpenho

OrdemBancaria
  → LiquidacaoEmpenho
```

## CRUDs Necessários

### Unidade Gestora (UG)

Representa órgãos, instituições e departamentos na hierarquia do sistema.

**Campos:**

- id
- codigoUg (código único que identifica a UG)
- nome
- orgaoSuperior (referência hierárquica, nullable para UGs raiz)
- status (ativo / inativo)
- descricao

**Operações:**

- Criar UG
- Consultar UG
- Atualizar UG
- Listar UGs
- Listar UGs subordinadas

**Exemplo:**
```
Ministério da Educação   (codigoUg: MIN_ED,  orgaoSuperior: null)
Secretaria de Ensino     (codigoUg: SES,     orgaoSuperior: MIN_ED)
Instituto X              (codigoUg: INST_X,  orgaoSuperior: SES)
```

---

### Classificações Orçamentárias

#### Ação Orçamentária

**Campos:** id, codigo, descricao, exercicio

CRUD completo.

#### Plano Interno (PI)

**Campos:** id, codigo, descricao, acaoOrcamentariaId

CRUD completo.

#### Natureza da Despesa (ND)

**Campos:** id, codigo, descricao, tipo

CRUD completo.

#### Fonte de Recurso

**Campos:** id, codigo, descricao

CRUD completo.

#### PTRES

**Campos:** id, codigo, descricao

CRUD completo.

---

### Dotação Orçamentária

Representa o orçamento disponível para uma UG em um exercício.

**Campos:**

- id
- ugId
- acaoOrcamentariaId
- planoInternoId
- naturezaDespesaId
- fonteRecursoId
- ptresId
- valorInicial
- valorAtualizado
- exercicio

**Operações:** Criar, Consultar, Atualizar, Consultar saldo.

**Saldo calculado:**
```
saldo = dotacao + creditos_recebidos - empenhos_emitidos
```

---

### Nota de Crédito (NC)

Usada para transferência de crédito orçamentário entre órgãos.

**Campos:**

- id
- numeroNc
- ugOrigem
- ugDestino
- dotacaoOrigem
- dotacaoDestino
- valor
- dataEmissao
- status

**Operações:** Criar, Aprovar, Cancelar, Consultar.

**Regras:**
- Valor da NC deve ser menor ou igual ao saldo disponível da dotação de origem.

---

### Fornecedor

**Campos:** id, cnpj, nome, tipoPessoa, status

**Operações:** Criar, Consultar, Atualizar, Pesquisar.

---

### Nota de Empenho (NE)

Reserva de orçamento para uma despesa.

**Campos:**

- id
- numeroEmpenho
- ugId
- dotacaoId
- fornecedorId
- naturezaDespesa
- planoInterno
- valor
- dataEmissao
- tipoEmpenho (ORDINARIO / ESTIMATIVO / GLOBAL)
- status

**Operações:** Criar, Anular, Consultar, Listar.

**Regras:**
- Valor do empenho deve ser menor ou igual ao saldo da dotação.

---

### Liquidação de Empenho (NL)

Confirma que o bem foi entregue ou o serviço foi prestado.

**Campos:**

- id
- numeroLiquidacao
- notaEmpenhoId
- valor
- dataLiquidacao
- documentoFiscal
- status

**Operações:** Criar, Consultar.

---

### Ordem Bancária (OB)

Registra o pagamento ao fornecedor.

**Campos:**

- id
- numeroOb
- liquidacaoId
- valor
- dataPagamento
- banco
- status

**Operações:** Criar, Consultar.
