# 06 — Regras de Negócio

## Princípios Gerais

### Imutabilidade de Documentos Financeiros

Documentos financeiros emitidos e contabilizados **não podem ser editados**. Qualquer correção exige cancelamento (ou estorno) do documento original e emissão de um novo documento correto. Esta é uma regra fundamental de contabilidade pública.

Exceção: campos meramente referenciais sem impacto contábil (ex.: número do processo administrativo, observações) podem ser alterados diretamente.

### Proibição de Exclusão Física

Nenhum documento financeiro pode ser excluído fisicamente do banco de dados. O registro permanece sempre visível e consultável, com seu status atualizado. Correções são feitas por estorno ou anulação, que geram contra-lançamentos mantendo o histórico completo.

### Numeração Automática de Documentos

Todo documento financeiro (NC, NE, NL, OB) recebe número gerado automaticamente pelo sistema no momento da emissão. O número é único por tipo de documento, UG e exercício financeiro. Números de documentos cancelados ou anulados não são reutilizados.

Formato: `[exercício (4 dígitos)][código UG][sequencial]`

### Integridade Transacional

Todas as operações que afetam saldos orçamentários devem ser ACID. Uma operação que falhe em qualquer etapa deve ser revertida completamente — nunca deixar saldos em estado inconsistente.

---

## Resumo das Ações por Entidade

| Entidade | Criar | Editar | Cancelar/Anular | Excluir |
|---|---|---|---|---|
| UnidadeGestora | sim | sim (campos não contábeis) | não (desativar) | não |
| Classificações Orçamentárias | sim | sim | não (desativar) | não se referenciada |
| DotacaoOrcamentaria | sim | não | não | não se tem empenhos |
| NotaCredito | sim | não | sim (somente PENDENTE) | não |
| Fornecedor | sim | sim | não (desativar) | não se referenciado em NE |
| NotaEmpenho | sim | não | sim (anulação total ou parcial) | não |
| LiquidacaoEmpenho | sim | não | sim (estorno) | não |
| OrdemBancaria | sim | não | sim (se não processada) | não |
| Usuario | sim | sim | não (desativar) | não |
| TokenIntegracao | sim | não | sim (revogar) | não |

---

## Dotação Orçamentária

### Ações

| Ação | Pré-condição | Efeito |
|---|---|---|
| Criar | UG ativa, classificações existentes | Cria dotação com `valorAtualizado = valorInicial` |
| Consultar saldo | — | Retorna saldo calculado |
| Crédito suplementar | Dotação existente, valor > 0 | Incrementa `valorAtualizado` |
| Desativar | Sem empenhos vinculados | Marca como inativa |

### Cálculo de Saldo

```
saldo_disponivel = valor_atualizado
                 + soma(NC aprovadas recebidas)
                 - soma(NC aprovadas cedidas)
                 - soma(NE emitidas não anuladas)
```

### Regras

- Não é possível criar NE ou aprovar NC cujo valor exceda o saldo disponível.
- Dotação com exercício diferente do exercício corrente é somente leitura para emissão de novos documentos.

---

## Nota de Crédito (NC)

### Ciclo de Vida

```
PENDENTE ──► APROVADA
    │
    └──────► CANCELADA
```

Após aprovada:

```
APROVADA ──► ESTORNADA  (somente se destino ainda não empenhrou os créditos)
```

### Estados

| Estado | Descrição |
|---|---|
| PENDENTE | NC registrada, ainda sem efeito nos saldos |
| APROVADA | Aprovada; saldos de origem e destino já impactados |
| CANCELADA | Cancelada antes da aprovação; sem efeito contábil |
| ESTORNADA | Aprovada e depois revertida; saldos restaurados |

### Ações

| Ação | Estado Atual | Pré-condição | Resultado |
|---|---|---|---|
| Criar NC | — | UG origem e destino ativas; valor ≤ saldo disponível da dotação origem | Status: PENDENTE |
| Aprovar NC | PENDENTE | — | Status: APROVADA; reduz saldo da origem, aumenta saldo do destino |
| Cancelar NC | PENDENTE | — | Status: CANCELADA; sem impacto em saldos |
| Estornar NC | APROVADA | Créditos recebidos pelo destino ainda não foram empenhados | Status: ESTORNADA; saldos restaurados |
| Consultar NC | qualquer | — | — |
| Listar NCs | — | — | Filtros: UG, exercício, status, período |

### Regras

- Uma NC não pode ser editada após criação; erros exigem cancelamento e nova NC.
- Não é possível estornar NC cujos créditos já tenham sido empenhados na UG destino. Os empenhos devem ser anulados primeiro.
- Estorno somente dentro do mesmo exercício financeiro.

---

## Nota de Empenho (NE)

### Ciclo de Vida

```
                    ┌──────────────────────────┐
                    ▼                          │ reforço
EMITIDA ──► A_LIQUIDAR ──► PARCIALMENTE_LIQUIDADA ──► LIQUIDADA ──► PAGA
               │                   │                      │
               └───────────────────┴──────────────────────┘
                                   │
                              ANULADA
```

### Estados

| Estado | Descrição |
|---|---|
| A_LIQUIDAR | Empenho emitido; nenhuma liquidação registrada |
| PARCIALMENTE_LIQUIDADA | Parte do valor foi liquidada (apenas ESTIMATIVO e GLOBAL) |
| LIQUIDADA | Valor total liquidado |
| PAGA | Valor total pago via OB |
| ANULADA | Cancelada total ou parcialmente |

### Ações

| Ação | Estado Atual | Pré-condição | Resultado |
|---|---|---|---|
| Criar NE | — | Dotação ativa; valor ≤ saldo disponível; fornecedor ativo | Status: A_LIQUIDAR; reserva saldo da dotação |
| Reforçar NE | A_LIQUIDAR ou PARCIALMENTE_LIQUIDADA | Valor do reforço ≤ saldo disponível da dotação; mesmo exercício | Aumenta valor do empenho; reduz saldo da dotação |
| Anular total | A_LIQUIDAR | Sem liquidações vigentes | Status: ANULADA; devolve saldo à dotação |
| Anular parcial | A_LIQUIDAR ou PARCIALMENTE_LIQUIDADA | Tipo: ESTIMATIVO ou GLOBAL; valor ≤ saldo a liquidar | Reduz valor do empenho; devolve diferença à dotação |
| Registrar liquidação | A_LIQUIDAR ou PARCIALMENTE_LIQUIDADA | Valor ≤ saldo a liquidar | Cria NL; atualiza status da NE |
| Consultar NE | qualquer | — | — |
| Listar NEs | — | — | Filtros: UG, dotação, fornecedor, status, tipo, período |

### Regras por Tipo

| Regra | ORDINARIO | ESTIMATIVO | GLOBAL |
|---|---|---|---|
| Anulação parcial de empenho | não | sim | sim |
| Liquidação parcial | não | sim | sim |
| Reforço | sim | sim | sim |
| Múltiplas liquidações | não | sim | sim |

### Regras Gerais

- NE de exercícios anteriores (Restos a Pagar) são somente leitura para criação; apenas anulação e consulta são permitidas.
- A anulação total só é possível se não houver liquidações vigentes. Liquidações existentes devem ser estornadas primeiro.
- Reforço somente dentro do mesmo exercício da NE original.
- O número do empenho é gerado automaticamente e não pode ser alterado.

---

## Liquidação de Empenho (NL)

### Ciclo de Vida

```
REGISTRADA ──► ESTORNADA
```

### Estados

| Estado | Descrição |
|---|---|
| REGISTRADA | Liquidação contabilizada; confirma recebimento do bem ou serviço |
| ESTORNADA | Liquidação revertida; valor retorna ao saldo a liquidar do empenho |

### Ações

| Ação | Estado Atual | Pré-condição | Resultado |
|---|---|---|---|
| Registrar liquidação | — | NE em A_LIQUIDAR ou PARCIALMENTE_LIQUIDADA; valor ≤ saldo a liquidar; documento fiscal informado | Cria NL com status REGISTRADA; atualiza NE |
| Estornar liquidação | REGISTRADA | Sem OB vigente vinculada | Status: ESTORNADA; devolve valor ao saldo a liquidar da NE |
| Consultar NL | qualquer | — | — |

### Regras

- Para NE do tipo ORDINARIO: apenas uma liquidação pelo valor total.
- Para NE do tipo ESTIMATIVO ou GLOBAL: múltiplas liquidações parciais são permitidas.
- A OB vinculada à NL deve ser cancelada antes do estorno da liquidação.
- Estorno somente dentro do mesmo exercício.
- Data da liquidação não pode ser anterior à data de emissão da NE.

---

## Ordem Bancária (OB)

### Ciclo de Vida

```
EMITIDA ──► PROCESSADA
   │
   └──────► CANCELADA
```

### Estados

| Estado | Descrição |
|---|---|
| EMITIDA | Autorizada no sistema; aguardando processamento bancário |
| PROCESSADA | Crédito efetivado na conta do fornecedor |
| CANCELADA | Não executada; recursos retornam ao disponível |

### Ações

| Ação | Estado Atual | Pré-condição | Resultado |
|---|---|---|---|
| Emitir OB | — | NL com status REGISTRADA; valor ≤ saldo liquidado não pago; fornecedor com dados bancários | Status: EMITIDA; reserva valor da liquidação |
| Processar OB | EMITIDA | — | Status: PROCESSADA; NL marcada como paga; NE atualizada |
| Cancelar OB | EMITIDA | — | Status: CANCELADA; valor retorna ao saldo a pagar da NL |
| Consultar OB | qualquer | — | — |

### Regras

- Uma NL pode ser paga por mais de uma OB (pagamentos parciais), desde que a soma das OBs não exceda o valor liquidado.
- OBs canceladas não podem ser reeditadas; deve-se emitir nova OB referenciando a mesma NL.
- Dados bancários do fornecedor (banco, agência, conta) são obrigatórios na emissão.

---

## Fornecedor

### Ações

| Ação | Pré-condição | Resultado |
|---|---|---|
| Cadastrar | CNPJ válido e único | Fornecedor ativo |
| Editar | Fornecedor existente | Atualiza dados cadastrais |
| Desativar | Sem NEs vigentes (não anuladas) vinculadas | Status: inativo |
| Pesquisar | — | Filtros: CNPJ, nome, status |

### Regras

- CNPJ deve ser único no sistema.
- Fornecedor inativo não pode ser vinculado a novas NEs.
- Não é possível excluir fornecedor referenciado em qualquer NE, ativa ou anulada.

---

## Unidade Gestora (UG)

### Ações

| Ação | Pré-condição | Resultado |
|---|---|---|
| Criar | Código único | UG ativa |
| Editar | — | Atualiza nome, descrição, orgaoSuperior |
| Desativar | Sem dotações ativas vinculadas | Status: inativo |
| Listar subordinadas | — | Retorna hierarquia a partir da UG informada |

### Regras

- Código da UG deve ser único no sistema.
- UG inativa não pode ser origem ou destino de novas NCs, nem ter novas dotações criadas.
- A alteração de `orgaoSuperior` não afeta documentos já emitidos.

---

## Classificações Orçamentárias

Aplicam-se igualmente a: AcaoOrcamentaria, PlanoInterno, NaturezaDespesa, FonteRecurso, PTRES.

### Ações

| Ação | Pré-condição | Resultado |
|---|---|---|
| Criar | Código único por tipo | Classificação ativa |
| Editar | — | Atualiza descrição e campos não identificadores |
| Desativar | — | Status: inativo; impede uso em novas dotações |
| Excluir | Nunca referenciada em dotações | Remove o registro |

### Regras

- Código deve ser único dentro do mesmo tipo de classificação.
- Classificação desativada não pode ser usada em novas dotações, mas dotações existentes não são afetadas.

---

## Usuário

### Ações

| Ação | Pré-condição | Resultado |
|---|---|---|
| Criar | Login único; senha válida conforme política | Usuário ativo |
| Editar | — | Atualiza nome, e-mail |
| Alterar senha | Senha atual correta ou token de redefinição válido | Nova senha criptografada armazenada |
| Desativar | — | Status: inativo; sessões existentes invalidadas |
| Listar | — | — |

### Regras

- Login deve ser único no sistema.
- Senhas devem ser armazenadas com hash (bcrypt, PBKDF2 ou Argon2).
- Após 5 tentativas de login inválidas consecutivas, a conta é bloqueada temporariamente.
- Usuário inativo não consegue autenticar.

---

## Token de Integração

### Ações

| Ação | Pré-condição | Resultado |
|---|---|---|
| Gerar | Usuário autenticado | Token criado com data de criação, expiração e status ATIVO |
| Listar | — | Retorna tokens do usuário autenticado (não exibe o valor do token, apenas metadados) |
| Revogar | Token existente e ATIVO | Status: REVOGADO; uso imediato impossibilitado |

### Regras

- O valor do token é exibido apenas uma vez, no momento da geração. Não é recuperável depois.
- Tokens expirados ou revogados são rejeitados em qualquer endpoint de API.
- Geração e revogação de tokens são registradas na auditoria.
- Um usuário pode ter múltiplos tokens ativos simultaneamente.

---

## Consultas e Relatórios

### Execução Orçamentária

Retorna, por dotação ou agrupamento, os seguintes valores:

| Campo | Origem |
|---|---|
| Dotação inicial | `DotacaoOrcamentaria.valorInicial` |
| Dotação atualizada | `DotacaoOrcamentaria.valorAtualizado` + NCs recebidas − NCs cedidas |
| Empenhado | Soma das NEs emitidas não anuladas |
| A liquidar | Empenhado − Liquidado |
| Liquidado | Soma das NLs registradas não estornadas |
| A pagar | Liquidado − Pago |
| Pago | Soma das OBs processadas |
| Saldo disponível | Dotação atualizada − Empenhado |

### Filtros obrigatórios suportados

UG, exercício, ação orçamentária, plano interno, natureza de despesa, fonte de recurso, PTRES, período (data inicial e final).
