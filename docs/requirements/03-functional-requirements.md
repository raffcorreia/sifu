# 03 — Requisitos Funcionais

## Consultas Financeiras

O sistema deve suportar as seguintes consultas:

- Saldo orçamentário por dotação
- Empenhos por período
- Empenhos liquidados
- Empenhos pagos
- Execução por ação orçamentária
- Execução por plano interno

**Filtros disponíveis:** UG, exercício, ação orçamentária, plano interno, natureza de despesa, fonte de recurso, PTRES, período.

## Dashboard

O dashboard deve exibir as seguintes métricas:

- Crédito disponível
- Total empenhado
- Total liquidado
- Total pago

## Requisitos Funcionais

| ID | Descrição |
|---|---|
| RF01 | Autenticação via tela de login com usuário e senha para todos os acessos. |
| RF02 | Geração de tokens de acesso (JWT ou similar) para uso por sistemas externos. |
| RF03 | Cadastro e manutenção de classificações orçamentárias. |
| RF04 | Criação e manutenção de dotações orçamentárias. |
| RF05 | Criação de Nota de Crédito. |
| RF06 | Aprovação ou cancelamento de Nota de Crédito. |
| RF07 | Criação de Nota de Empenho. |
| RF08 | Validação de saldo antes de emitir empenho. |
| RF09 | Liquidação de empenho. |
| RF10 | Registro de pagamento via Ordem Bancária. |
| RF11 | Consultas de execução orçamentária com filtros. |
| RF12 | Geração de dashboards financeiros. |
| RF13 | API exposta para integração externa com geração e revogação de tokens. |
| RF14 | Tela de login com suporte a logout, recuperação de senha e validação de força de senha. |
| RF15 | Todos os usuários têm privilégios administrativos completos (simplificação educacional). |
| RF16 | Usuários autenticados podem gerar, listar e revogar tokens de integração. |
| RF17 | Todo token de integração deve estar vinculado ao usuário emissor e registrar data de criação, expiração e status. |
| RF18 | A aplicação deve expor API versionada para integração externa de todas as operações no escopo. |
| RF19 | A API deve ter documentação funcional dos endpoints, parâmetros, payloads, códigos de resposta e exemplos de uso. |
