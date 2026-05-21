# 13 — Processo de QA e Aceite por Fase

> **Camada:** Operations | **Depende de:** 09-testing

## Princípio

Cada fase do projeto só é considerada **concluída** após:

1. CI verde (build + testes + cobertura)
2. Testes automatizados escritos e cobertos conforme 09-testing.md
3. **QA executa os casos de teste da fase e assina o aceite**

Nenhuma fase avança sem o sign-off do QA. O QA não testa o que não está documentado — os critérios de aceite devem existir antes do início da fase.

---

## Fluxo de uma Fase

```
┌─────────────────────────────────────────────────────┐
│  1. PLANEJAMENTO                                     │
│     - Funcionalidades da fase definidas              │
│     - Critérios de aceite escritos (este doc)        │
│     - Casos de teste de QA listados (esta seção)     │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  2. DESENVOLVIMENTO                                  │
│     - Código implementado                           │
│     - Testes automatizados escritos junto ao código  │
│     - CI passando                                    │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  3. QA — VALIDAÇÃO                                  │
│     - QA executa os casos de teste manuais           │
│     - QA verifica os critérios de aceite             │
│     - QA registra resultado: APROVADO ou REPROVADO   │
└──────────────────────┬──────────────────────────────┘
                       ↓
         ┌─────────────┴─────────────┐
         ▼                           ▼
    APROVADO                    REPROVADO
    Fase concluída              Bugs registrados
    Avança para a próxima       Desenvolvimento corrige
                                Volta para QA
```

---

## Estrutura dos Casos de Teste de QA

Cada fase tem seus casos de teste listados abaixo. Cada caso deve ser executado manualmente pelo QA no ambiente de homologação.

### Formato de um Caso de Teste

```
ID:          [FASE]-[NUMERO]
Título:      Descrição curta
Pré-condição: O que precisa existir antes de executar
Passos:      1. Ação do usuário
             2. Ação do usuário
             ...
Resultado Esperado: O que deve acontecer
Resultado Obtido:   (preenchido pelo QA)
Status:      PASSOU / FALHOU
```

---

## Casos de Teste por Fase

> **Nota:** As fases precisam ser definidas no planejamento do projeto. A estrutura abaixo serve como referência para quando as fases forem formalizadas. Os casos de teste de cada fase devem ser escritos **antes** do início da implementação.

### FASE 1 — Setup Técnico e Autenticação

**Critério de aceite:** Sistema rodando localmente via Docker. Login e logout funcionando. Usuário admin criado via migration.

| ID | Título |
|---|---|
| F1-01 | Ambiente sobe com `docker compose up` sem erros |
| F1-02 | Login com credenciais válidas retorna token |
| F1-03 | Login com senha incorreta retorna erro 401 |
| F1-04 | Conta bloqueada após 5 tentativas inválidas |
| F1-05 | Logout invalida a sessão |
| F1-06 | Recuperação de senha envia e-mail |
| F1-07 | Redefinição de senha com token válido funciona |
| F1-08 | Redefinição com token expirado retorna erro |
| F1-09 | Swagger UI acessível e documentado |
| F1-10 | Requisição sem token retorna 401 |

### FASE 2 — Estrutura Organizacional e Classificações

**Critério de aceite:** É possível cadastrar e manter UGs, hierarquia e todas as classificações orçamentárias.

| ID | Título |
|---|---|
| F2-01 | Cadastrar UG raiz (sem órgão superior) |
| F2-02 | Cadastrar UG subordinada e exibir hierarquia |
| F2-03 | Editar UG e verificar alteração |
| F2-04 | Desativar UG e verificar que não aparece em novos cadastros |
| F2-05 | CRUD completo de Ação Orçamentária |
| F2-06 | CRUD completo de Plano Interno vinculado a Ação |
| F2-07 | CRUD completo de Natureza de Despesa |
| F2-08 | CRUD completo de Fonte de Recurso |
| F2-09 | CRUD completo de PTRES |
| F2-10 | Tentativa de excluir classificação referenciada retorna erro |

### FASE 3 — Dotações e Notas de Crédito

**Critério de aceite:** É possível criar dotações e transferir créditos entre UGs via NC.

| ID | Título |
|---|---|
| F3-01 | Criar dotação e consultar saldo inicial |
| F3-02 | Aplicar crédito suplementar e verificar saldo atualizado |
| F3-03 | Criar NC entre duas UGs |
| F3-04 | Aprovar NC e verificar saldo reduzido na origem e aumentado no destino |
| F3-05 | Cancelar NC ainda PENDENTE |
| F3-06 | Tentativa de aprovar NC já CANCELADA retorna erro |
| F3-07 | Criar NC com valor superior ao saldo disponível retorna erro |
| F3-08 | Estornar NC APROVADA cujos créditos não foram empenhados |
| F3-09 | Tentativa de estornar NC com créditos já empenhados retorna erro |
| F3-10 | Filtros de listagem de NC funcionam (por UG, status, período) |

### FASE 4 — Fornecedores e Notas de Empenho

**Critério de aceite:** É possível cadastrar fornecedores e emitir, reforçar e anular empenhos.

| ID | Título |
|---|---|
| F4-01 | Cadastrar fornecedor com CNPJ válido |
| F4-02 | Tentativa de cadastrar CNPJ duplicado retorna erro |
| F4-03 | Desativar fornecedor e verificar que não aparece em nova NE |
| F4-04 | Emitir NE ORDINARIO com saldo suficiente |
| F4-05 | Tentativa de emitir NE com saldo insuficiente retorna erro |
| F4-06 | Emitir NE ESTIMATIVO e verificar tipo |
| F4-07 | Emitir NE GLOBAL e verificar tipo |
| F4-08 | Reforçar NE e verificar novo valor e redução do saldo |
| F4-09 | Anular NE ORDINARIO totalmente |
| F4-10 | Tentativa de anular NE ORDINARIO parcialmente retorna erro |
| F4-11 | Anular NE ESTIMATIVO parcialmente |
| F4-12 | Tentativa de anular NE com liquidação vigente retorna erro |
| F4-13 | Filtros de listagem de NE funcionam (UG, status, tipo, período) |

### FASE 5 — Liquidação e Ordem Bancária

**Critério de aceite:** É possível liquidar empenhos e registrar pagamentos. O ciclo completo funciona de ponta a ponta.

| ID | Título |
|---|---|
| F5-01 | Registrar liquidação total de NE ORDINARIO |
| F5-02 | Registrar liquidação parcial de NE ESTIMATIVO |
| F5-03 | Tentativa de liquidação parcial de NE ORDINARIO retorna erro |
| F5-04 | Tentativa de liquidar acima do saldo a liquidar retorna erro |
| F5-05 | Estornar liquidação sem OB vinculada |
| F5-06 | Tentativa de estornar liquidação com OB retorna erro |
| F5-07 | Emitir OB para liquidação registrada |
| F5-08 | Processar OB e verificar status da NE como PAGA |
| F5-09 | Cancelar OB EMITIDA e verificar retorno do saldo |
| F5-10 | Fluxo completo: Dotação → NC → NE → NL → OB |

### FASE 6 — Consultas, Dashboard e API de Integração

**Critério de aceite:** Consultas e dashboard refletem dados corretos. API de integração funciona com token.

| ID | Título |
|---|---|
| F6-01 | Dashboard exibe crédito disponível, empenhado, liquidado e pago corretos |
| F6-02 | Consulta de execução orçamentária com filtro por UG e exercício |
| F6-03 | Filtros de execução por ação orçamentária e plano interno |
| F6-04 | Gerar token de integração e exibir uma única vez |
| F6-05 | Usar token de integração para consultar NEs via API |
| F6-06 | Revogar token e verificar que requisições retornam 401 |
| F6-07 | Token expirado retorna 401 |
| F6-08 | Rate limit de 100 req/min é respeitado |
| F6-09 | Consulta de auditoria exibe todas as operações da sessão |

---

## Registro de Sign-off

Ao concluir a validação de uma fase, o QA preenche:

```
Fase:           [FASE 1 / FASE 2 / ...]
Data:           YYYY-MM-DD
QA responsável: [nome]
Total de casos: [N]
Passou:         [N]
Falhou:         [N]
Bugs registrados: [link(s) para issues]
Decisão:        APROVADO / REPROVADO
Observações:    [campo livre]
```

---

## Critérios de Reprovação Automática

A fase é automaticamente **reprovada** (sem necessidade de executar todos os casos) se:

- CI com falha no momento do QA
- Cobertura de testes abaixo do mínimo definido em 09-testing.md
- Qualquer caso de teste de fluxo crítico (NC, NE, NL, OB, auth) falhar
- Dados corrompidos ou inconsistências de saldo encontradas
