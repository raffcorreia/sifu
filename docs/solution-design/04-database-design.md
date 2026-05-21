# 04 — Design do Banco de Dados

> **Camada:** Core Design | **Depende de:** 03-architecture

## Convenções

- Nomes de tabelas: `snake_case`, plural
- Nomes de colunas: `snake_case`
- Chaves primárias: `BIGSERIAL` com nome `id`
- Timestamps: `TIMESTAMPTZ` (com fuso horário) em todas as tabelas
- Valores monetários: `NUMERIC(15,2)` — escala suficiente para orçamentos públicos
- Status: `VARCHAR` com `CHECK` constraint — evita tabelas de lookup para valores simples
- Exclusão: nunca física; campo `status` com valor `INATIVO` ou `ANULADA` etc.

## Schema Completo

```sql
-- ============================================================
-- SEGURANÇA
-- ============================================================

CREATE TABLE usuarios (
    id            BIGSERIAL PRIMARY KEY,
    login         VARCHAR(50)  NOT NULL UNIQUE,
    nome          VARCHAR(200) NOT NULL,
    email         VARCHAR(200) NOT NULL UNIQUE,
    senha_hash    VARCHAR(200) NOT NULL,
    tentativas_login  INTEGER  NOT NULL DEFAULT 0,
    bloqueado_ate TIMESTAMPTZ,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE tokens_integracao (
    id              BIGSERIAL PRIMARY KEY,
    usuario_id      BIGINT       NOT NULL REFERENCES usuarios(id),
    nome            VARCHAR(100) NOT NULL,
    token_hash      VARCHAR(200) NOT NULL UNIQUE,
    data_criacao    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    data_expiracao  TIMESTAMPTZ  NOT NULL,
    status          VARCHAR(15)  NOT NULL DEFAULT 'ATIVO'
                    CHECK (status IN ('ATIVO', 'REVOGADO', 'EXPIRADO')),
    criado_em       TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE tokens_redefinicao_senha (
    id          BIGSERIAL    PRIMARY KEY,
    usuario_id  BIGINT       NOT NULL REFERENCES usuarios(id),
    token       VARCHAR(100) NOT NULL UNIQUE,
    expira_em   TIMESTAMPTZ  NOT NULL,
    usado       BOOLEAN      NOT NULL DEFAULT FALSE,
    criado_em   TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- ESTRUTURA ORGANIZACIONAL
-- ============================================================

CREATE TABLE unidades_gestoras (
    id                BIGSERIAL PRIMARY KEY,
    codigo_ug         VARCHAR(20)  NOT NULL UNIQUE,
    nome              VARCHAR(200) NOT NULL,
    orgao_superior_id BIGINT       REFERENCES unidades_gestoras(id),
    descricao         TEXT,
    status            VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                      CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em         TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- CLASSIFICAÇÕES ORÇAMENTÁRIAS
-- ============================================================

CREATE TABLE acoes_orcamentarias (
    id            BIGSERIAL PRIMARY KEY,
    codigo        VARCHAR(20)  NOT NULL,
    descricao     VARCHAR(300) NOT NULL,
    exercicio     INTEGER      NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    UNIQUE (codigo, exercicio)
);

CREATE TABLE planos_internos (
    id                    BIGSERIAL PRIMARY KEY,
    codigo                VARCHAR(20)  NOT NULL UNIQUE,
    descricao             VARCHAR(300) NOT NULL,
    acao_orcamentaria_id  BIGINT       NOT NULL REFERENCES acoes_orcamentarias(id),
    status                VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                          CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em             TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em         TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE naturezas_despesa (
    id            BIGSERIAL PRIMARY KEY,
    codigo        VARCHAR(20)  NOT NULL UNIQUE,
    descricao     VARCHAR(300) NOT NULL,
    tipo          VARCHAR(50),
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE fontes_recurso (
    id            BIGSERIAL PRIMARY KEY,
    codigo        VARCHAR(20)  NOT NULL UNIQUE,
    descricao     VARCHAR(300) NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE ptres (
    id            BIGSERIAL PRIMARY KEY,
    codigo        VARCHAR(20)  NOT NULL UNIQUE,
    descricao     VARCHAR(300) NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- ORÇAMENTO
-- ============================================================

CREATE TABLE dotacoes_orcamentarias (
    id                    BIGSERIAL       PRIMARY KEY,
    ug_id                 BIGINT          NOT NULL REFERENCES unidades_gestoras(id),
    acao_orcamentaria_id  BIGINT          NOT NULL REFERENCES acoes_orcamentarias(id),
    plano_interno_id      BIGINT          REFERENCES planos_internos(id),
    natureza_despesa_id   BIGINT          NOT NULL REFERENCES naturezas_despesa(id),
    fonte_recurso_id      BIGINT          NOT NULL REFERENCES fontes_recurso(id),
    ptres_id              BIGINT          NOT NULL REFERENCES ptres(id),
    valor_inicial         NUMERIC(15,2)   NOT NULL CHECK (valor_inicial >= 0),
    valor_atualizado      NUMERIC(15,2)   NOT NULL CHECK (valor_atualizado >= 0),
    exercicio             INTEGER         NOT NULL,
    criado_em             TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    atualizado_em         TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);

CREATE TABLE notas_credito (
    id               BIGSERIAL     PRIMARY KEY,
    numero_nc        VARCHAR(20)   NOT NULL UNIQUE,
    ug_origem_id     BIGINT        NOT NULL REFERENCES unidades_gestoras(id),
    ug_destino_id    BIGINT        NOT NULL REFERENCES unidades_gestoras(id),
    dotacao_origem_id  BIGINT      NOT NULL REFERENCES dotacoes_orcamentarias(id),
    dotacao_destino_id BIGINT      NOT NULL REFERENCES dotacoes_orcamentarias(id),
    valor            NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    data_emissao     DATE          NOT NULL,
    status           VARCHAR(20)   NOT NULL DEFAULT 'PENDENTE'
                     CHECK (status IN ('PENDENTE', 'APROVADA', 'CANCELADA', 'ESTORNADA')),
    criado_em        TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    atualizado_em    TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    CHECK (ug_origem_id <> ug_destino_id)
);

-- ============================================================
-- EXECUÇÃO
-- ============================================================

CREATE TABLE fornecedores (
    id            BIGSERIAL    PRIMARY KEY,
    cnpj          VARCHAR(14)  NOT NULL UNIQUE,
    nome          VARCHAR(200) NOT NULL,
    tipo_pessoa   VARCHAR(10)  NOT NULL CHECK (tipo_pessoa IN ('JURIDICA', 'FISICA')),
    banco         VARCHAR(10),
    agencia       VARCHAR(10),
    conta_corrente VARCHAR(20),
    status        VARCHAR(10)  NOT NULL DEFAULT 'ATIVO'
                  CHECK (status IN ('ATIVO', 'INATIVO')),
    criado_em     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE notas_empenho (
    id               BIGSERIAL     PRIMARY KEY,
    numero_empenho   VARCHAR(20)   NOT NULL UNIQUE,
    ug_id            BIGINT        NOT NULL REFERENCES unidades_gestoras(id),
    dotacao_id       BIGINT        NOT NULL REFERENCES dotacoes_orcamentarias(id),
    fornecedor_id    BIGINT        NOT NULL REFERENCES fornecedores(id),
    valor            NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    data_emissao     DATE          NOT NULL,
    tipo_empenho     VARCHAR(15)   NOT NULL
                     CHECK (tipo_empenho IN ('ORDINARIO', 'ESTIMATIVO', 'GLOBAL')),
    status           VARCHAR(30)   NOT NULL DEFAULT 'A_LIQUIDAR'
                     CHECK (status IN ('A_LIQUIDAR', 'PARCIALMENTE_LIQUIDADA', 'LIQUIDADA', 'PAGA', 'ANULADA')),
    processo         VARCHAR(50),
    descricao_objeto TEXT,
    criado_em        TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    atualizado_em    TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

CREATE TABLE liquidacoes_empenho (
    id                 BIGSERIAL     PRIMARY KEY,
    numero_liquidacao  VARCHAR(20)   NOT NULL UNIQUE,
    nota_empenho_id    BIGINT        NOT NULL REFERENCES notas_empenho(id),
    valor              NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    data_liquidacao    DATE          NOT NULL,
    documento_fiscal   VARCHAR(50)   NOT NULL,
    status             VARCHAR(20)   NOT NULL DEFAULT 'REGISTRADA'
                       CHECK (status IN ('REGISTRADA', 'ESTORNADA')),
    criado_em          TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    atualizado_em      TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

CREATE TABLE ordens_bancarias (
    id             BIGSERIAL     PRIMARY KEY,
    numero_ob      VARCHAR(20)   NOT NULL UNIQUE,
    liquidacao_id  BIGINT        NOT NULL REFERENCES liquidacoes_empenho(id),
    valor          NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    data_pagamento DATE,
    banco          VARCHAR(10)   NOT NULL,
    agencia        VARCHAR(10)   NOT NULL,
    conta_destino  VARCHAR(20)   NOT NULL,
    status         VARCHAR(20)   NOT NULL DEFAULT 'EMITIDA'
                   CHECK (status IN ('EMITIDA', 'PROCESSADA', 'CANCELADA')),
    criado_em      TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    atualizado_em  TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

-- ============================================================
-- AUDITORIA
-- ============================================================

CREATE TABLE auditoria (
    id            BIGSERIAL    PRIMARY KEY,
    usuario_login VARCHAR(50)  NOT NULL,
    data_hora     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    operacao      VARCHAR(100) NOT NULL,
    entidade      VARCHAR(100) NOT NULL,
    entidade_id   VARCHAR(50),
    dados_antes   JSONB,
    dados_depois  JSONB,
    ip            VARCHAR(45)  NOT NULL,
    ug_id         BIGINT       REFERENCES unidades_gestoras(id)
);
```

## Índices

```sql
-- Buscas frequentes por código
CREATE INDEX idx_ug_codigo           ON unidades_gestoras(codigo_ug);
CREATE INDEX idx_ug_superior         ON unidades_gestoras(orgao_superior_id);
CREATE INDEX idx_fornecedor_cnpj     ON fornecedores(cnpj);
CREATE INDEX idx_ne_numero           ON notas_empenho(numero_empenho);
CREATE INDEX idx_ne_dotacao          ON notas_empenho(dotacao_id);
CREATE INDEX idx_ne_status           ON notas_empenho(status);
CREATE INDEX idx_ne_ug_exercicio     ON notas_empenho(ug_id, data_emissao);
CREATE INDEX idx_nc_numero           ON notas_credito(numero_nc);
CREATE INDEX idx_nc_status           ON notas_credito(status);
CREATE INDEX idx_nl_ne               ON liquidacoes_empenho(nota_empenho_id);
CREATE INDEX idx_ob_liquidacao       ON ordens_bancarias(liquidacao_id);

-- Auditoria: consultas por entidade e período
CREATE INDEX idx_auditoria_entidade  ON auditoria(entidade, entidade_id);
CREATE INDEX idx_auditoria_usuario   ON auditoria(usuario_login);
CREATE INDEX idx_auditoria_data      ON auditoria(data_hora DESC);

-- Consulta de execução orçamentária (filtros compostos frequentes)
CREATE INDEX idx_dotacao_ug_exercicio ON dotacoes_orcamentarias(ug_id, exercicio);
```

## Numeração de Documentos

A numeração automática (NC, NE, NL, OB) é gerada pela aplicação, não pelo banco. Formato:

```
[exercício 4 dig][código UG sem traços][sequencial 6 dig]
Exemplo NE: 2025MIN_ED000001
```

A sequência é mantida por uma tabela de controle:

```sql
CREATE TABLE sequencias_documentos (
    tipo_documento  VARCHAR(5)   NOT NULL,
    ug_id           BIGINT       NOT NULL REFERENCES unidades_gestoras(id),
    exercicio       INTEGER      NOT NULL,
    ultimo_numero   INTEGER      NOT NULL DEFAULT 0,
    PRIMARY KEY (tipo_documento, ug_id, exercicio)
);
```

O incremento usa `SELECT ... FOR UPDATE` na mesma transação da emissão do documento — garante unicidade sem concorrência.

## Migrações (Flyway)

Localização: `src/main/resources/db/migration/`

```
V1__criar_tabelas_seguranca.sql
V2__criar_tabelas_estrutura.sql
V3__criar_tabelas_classificacoes.sql
V4__criar_tabelas_orcamento.sql
V5__criar_tabelas_execucao.sql
V6__criar_tabela_auditoria.sql
V7__criar_indices.sql
V8__dados_iniciais.sql       ← usuário admin padrão, UG raiz de exemplo
```
