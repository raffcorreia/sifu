# 04 — Design do Banco de Dados

> **Camada:** Core Design | **Depende de:** 03-architecture

## Convenções

- Nomes de tabelas: `snake_case`, plural
- Nomes de colunas: `snake_case`
- Chaves primárias: `BIGSERIAL` com nome `id`
- Timestamps: `TIMESTAMPTZ` (com fuso horário) em todas as tabelas
- Valores monetários: `NUMERIC(15,2)` — escala suficiente para orçamentos públicos
- Status: `VARCHAR` com `CHECK` constraint — evita tabelas de lookup para valores simples; valores de status são strings em inglês (ex: `'ACTIVE'`, `'CANCELLED'`)
- Exclusão: nunca física; campo `status` com valor `'INACTIVE'` ou `'VOIDED'` etc.

## Schema Completo

```sql
-- ============================================================
-- SEGURANÇA
-- ============================================================

CREATE TABLE users (
    id            BIGSERIAL PRIMARY KEY,
    login         VARCHAR(50)  NOT NULL UNIQUE,
    name          VARCHAR(200) NOT NULL,
    email         VARCHAR(200) NOT NULL UNIQUE,
    password_hash VARCHAR(200) NOT NULL,
    login_attempts    INTEGER  NOT NULL DEFAULT 0,
    locked_until  TIMESTAMPTZ,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                  CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE integration_tokens (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT       NOT NULL REFERENCES users(id),
    name            VARCHAR(100) NOT NULL,
    token_hash      VARCHAR(200) NOT NULL UNIQUE,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    expires_at      TIMESTAMPTZ  NOT NULL,
    status          VARCHAR(15)  NOT NULL DEFAULT 'ACTIVE'
                    CHECK (status IN ('ACTIVE', 'REVOKED', 'EXPIRED'))
);

CREATE TABLE password_reset_tokens (
    id          BIGSERIAL    PRIMARY KEY,
    user_id     BIGINT       NOT NULL REFERENCES users(id),
    token       VARCHAR(100) NOT NULL UNIQUE,
    expira_em   TIMESTAMPTZ  NOT NULL,
    usado       BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- ESTRUTURA ORGANIZACIONAL
-- ============================================================

CREATE TABLE managing_units (
    id                BIGSERIAL PRIMARY KEY,
    unit_code         VARCHAR(20)  NOT NULL UNIQUE,
    name              VARCHAR(200) NOT NULL,
    parent_unit_id    BIGINT       REFERENCES managing_units(id),
    description       TEXT,
    status            VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                      CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at        TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at        TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- CLASSIFICAÇÕES ORÇAMENTÁRIAS
-- ============================================================

CREATE TABLE budget_actions (
    id            BIGSERIAL PRIMARY KEY,
    code          VARCHAR(20)  NOT NULL,
    description   VARCHAR(300) NOT NULL,
    fiscal_year   INTEGER      NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                  CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    UNIQUE (code, fiscal_year)
);

CREATE TABLE internal_plans (
    id                    BIGSERIAL PRIMARY KEY,
    code                  VARCHAR(20)  NOT NULL UNIQUE,
    description           VARCHAR(300) NOT NULL,
    budget_action_id      BIGINT       NOT NULL REFERENCES budget_actions(id),
    status                VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                          CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at            TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at            TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE expense_natures (
    id            BIGSERIAL PRIMARY KEY,
    code          VARCHAR(20)  NOT NULL UNIQUE,
    description   VARCHAR(300) NOT NULL,
    tipo          VARCHAR(50),
    status        VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                  CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE funding_sources (
    id            BIGSERIAL PRIMARY KEY,
    code          VARCHAR(20)  NOT NULL UNIQUE,
    description   VARCHAR(300) NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                  CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE ptres (
    id            BIGSERIAL PRIMARY KEY,
    code          VARCHAR(20)  NOT NULL UNIQUE,
    description   VARCHAR(300) NOT NULL,
    status        VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                  CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- ============================================================
-- ORÇAMENTO
-- ============================================================

CREATE TABLE budget_allotments (
    id                    BIGSERIAL       PRIMARY KEY,
    unit_id               BIGINT          NOT NULL REFERENCES managing_units(id),
    budget_action_id      BIGINT          NOT NULL REFERENCES budget_actions(id),
    internal_plan_id      BIGINT          REFERENCES internal_plans(id),
    expense_nature_id     BIGINT          NOT NULL REFERENCES expense_natures(id),
    funding_source_id     BIGINT          NOT NULL REFERENCES funding_sources(id),
    ptres_id              BIGINT          NOT NULL REFERENCES ptres(id),
    initial_value         NUMERIC(15,2)   NOT NULL CHECK (initial_value >= 0),
    updated_value         NUMERIC(15,2)   NOT NULL CHECK (updated_value >= 0),
    fiscal_year           INTEGER         NOT NULL,
    created_at            TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at            TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);

CREATE TABLE credit_notes (
    id                   BIGSERIAL     PRIMARY KEY,
    credit_note_number   VARCHAR(20)   NOT NULL UNIQUE,
    source_unit_id       BIGINT        NOT NULL REFERENCES managing_units(id),
    target_unit_id       BIGINT        NOT NULL REFERENCES managing_units(id),
    source_allotment_id  BIGINT        NOT NULL REFERENCES budget_allotments(id),
    target_allotment_id  BIGINT        NOT NULL REFERENCES budget_allotments(id),
    valor                NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    issue_date           DATE          NOT NULL,
    status               VARCHAR(20)   NOT NULL DEFAULT 'PENDING'
                         CHECK (status IN ('PENDING', 'APPROVED', 'CANCELLED', 'REVERSED')),
    created_at           TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_at           TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    CHECK (source_unit_id <> target_unit_id)
);

-- ============================================================
-- EXECUÇÃO
-- ============================================================

CREATE TABLE vendors (
    id             BIGSERIAL    PRIMARY KEY,
    cnpj           VARCHAR(14)  NOT NULL UNIQUE,
    name           VARCHAR(200) NOT NULL,
    person_type    VARCHAR(15)  NOT NULL CHECK (person_type IN ('LEGAL_ENTITY', 'INDIVIDUAL')),
    bank           VARCHAR(10),
    branch         VARCHAR(10),
    account_number VARCHAR(20),
    status         VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                   CHECK (status IN ('ACTIVE', 'INACTIVE')),
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE commitments (
    id                  BIGSERIAL     PRIMARY KEY,
    commitment_number   VARCHAR(20)   NOT NULL UNIQUE,
    unit_id             BIGINT        NOT NULL REFERENCES managing_units(id),
    allotment_id        BIGINT        NOT NULL REFERENCES budget_allotments(id),
    vendor_id           BIGINT        NOT NULL REFERENCES vendors(id),
    valor               NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    issue_date          DATE          NOT NULL,
    commitment_type     VARCHAR(15)   NOT NULL
                        CHECK (commitment_type IN ('ORDINARY', 'ESTIMATED', 'GLOBAL')),
    status              VARCHAR(30)   NOT NULL DEFAULT 'PENDING_SETTLEMENT'
                        CHECK (status IN ('PENDING_SETTLEMENT', 'PARTIALLY_SETTLED', 'SETTLED', 'PAID', 'VOIDED')),
    process_number      VARCHAR(50),
    object_description  TEXT,
    created_at          TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

CREATE TABLE settlements (
    id                 BIGSERIAL     PRIMARY KEY,
    settlement_number  VARCHAR(20)   NOT NULL UNIQUE,
    commitment_id      BIGINT        NOT NULL REFERENCES commitments(id),
    valor              NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    settlement_date    DATE          NOT NULL,
    fiscal_document    VARCHAR(50)   NOT NULL,
    status             VARCHAR(20)   NOT NULL DEFAULT 'REGISTERED'
                       CHECK (status IN ('REGISTERED', 'REVERSED')),
    created_at         TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

CREATE TABLE payment_orders (
    id                   BIGSERIAL     PRIMARY KEY,
    payment_order_number VARCHAR(20)   NOT NULL UNIQUE,
    settlement_id        BIGINT        NOT NULL REFERENCES settlements(id),
    valor                NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    payment_date         DATE,
    bank                 VARCHAR(10)   NOT NULL,
    branch               VARCHAR(10)   NOT NULL,
    destination_account  VARCHAR(20)   NOT NULL,
    status               VARCHAR(20)   NOT NULL DEFAULT 'ISSUED'
                         CHECK (status IN ('ISSUED', 'PROCESSED', 'CANCELLED')),
    created_at           TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_at           TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

-- ============================================================
-- AUDITORIA
-- ============================================================

CREATE TABLE audit_log (
    id            BIGSERIAL    PRIMARY KEY,
    user_login    VARCHAR(50)  NOT NULL,
    event_time    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    operation     VARCHAR(100) NOT NULL,
    entity        VARCHAR(100) NOT NULL,
    entity_id     VARCHAR(50),
    data_before   JSONB,
    data_after    JSONB,
    ip_address    VARCHAR(45)  NOT NULL,
    unit_id       BIGINT       REFERENCES managing_units(id)
);
```

## Índices

```sql
-- Buscas frequentes por código
CREATE INDEX idx_unit_code              ON managing_units(unit_code);
CREATE INDEX idx_unit_parent            ON managing_units(parent_unit_id);
CREATE INDEX idx_vendor_cnpj            ON vendors(cnpj);
CREATE INDEX idx_commitment_number      ON commitments(commitment_number);
CREATE INDEX idx_commitment_allotment   ON commitments(allotment_id);
CREATE INDEX idx_commitment_status      ON commitments(status);
CREATE INDEX idx_commitment_unit_year   ON commitments(unit_id, issue_date);
CREATE INDEX idx_credit_note_number     ON credit_notes(credit_note_number);
CREATE INDEX idx_credit_note_status     ON credit_notes(status);
CREATE INDEX idx_settlement_commitment  ON settlements(commitment_id);
CREATE INDEX idx_payment_order_settlement ON payment_orders(settlement_id);

-- Auditoria: consultas por entidade e período
CREATE INDEX idx_audit_entity  ON audit_log(entity, entity_id);
CREATE INDEX idx_audit_user    ON audit_log(user_login);
CREATE INDEX idx_audit_time    ON audit_log(event_time DESC);

-- Consulta de execução orçamentária (filtros compostos frequentes)
CREATE INDEX idx_allotment_unit_year ON budget_allotments(unit_id, fiscal_year);
```

## Numeração de Documentos

A numeração automática (NC, NE, NL, OB) é gerada pela aplicação, não pelo banco. Formato:

```
[exercício 4 dig][código UG sem traços][sequencial 6 dig]
Exemplo NE: 2025MIN_ED000001
```

A sequência é mantida por uma tabela de controle:

```sql
CREATE TABLE document_sequences (
    document_type   VARCHAR(5)   NOT NULL,
    unit_id         BIGINT       NOT NULL REFERENCES managing_units(id),
    fiscal_year     INTEGER      NOT NULL,
    last_number     INTEGER      NOT NULL DEFAULT 0,
    PRIMARY KEY (document_type, unit_id, fiscal_year)
);
```

O incremento usa `SELECT ... FOR UPDATE` na mesma transação da emissão do documento — garante unicidade sem concorrência.

## Migrações (Flyway)

Localização: `src/main/resources/db/migration/`

```
V1__create_security_tables.sql
V2__create_structure_tables.sql
V3__create_classification_tables.sql
V4__create_budget_tables.sql
V5__create_execution_tables.sql
V6__create_audit_table.sql
V7__create_indexes.sql
V8__initial_data.sql       ← usuário admin padrão, UG raiz de exemplo
```
