# 04 — Histórias de Usuário

## Como Operador

- Como operador, quero cadastrar classificações orçamentárias para organizar o orçamento da instituição.
- Como operador, quero criar dotações orçamentárias para definir e acompanhar os recursos disponíveis.
- Como operador, quero emitir Nota de Crédito para transferir recursos entre órgãos e departamentos.
- Como operador, quero criar Nota de Empenho para reservar orçamento para futuras despesas.
- Como operador, quero liquidar empenhos após confirmação de recebimento de bens ou serviços.
- Como operador, quero registrar pagamentos via Ordem Bancária para efetuar o desembolso.
- Como operador, quero fazer login seguro no sistema usando usuário e senha.

## Como Gestor Financeiro

- Como gestor financeiro, quero revisar e aprovar Notas de Crédito entre órgãos.
- Como gestor financeiro, quero consultar saldos orçamentários em tempo real.
- Como gestor financeiro, quero visualizar dashboards de execução orçamentária.
- Como gestor financeiro, quero monitorar o status de todas as operações financeiras.

## Como Ordenador de Despesas

- Como ordenador, quero validar disponibilidade orçamentária antes de emitir empenhos.
- Como ordenador, quero consultar empenhos em diferentes estados (emitidos, liquidados, pagos).
- Como ordenador, quero gerar relatórios de execução de despesas.

## Como Administrador

- Como administrador, quero gerenciar usuários e suas credenciais de acesso.
- Como administrador, quero revisar logs completos de auditoria para rastrear todas as operações.
- Como administrador, quero cadastrar e manter a hierarquia de órgãos e unidades gestoras.
- Como administrador, quero gerar e revogar tokens de acesso para que outros sistemas possam integrar com o SIFU.

## Histórias Técnicas

- Como desenvolvedor, quero configurar o banco de dados PostgreSQL e aplicar migrations.
- Como desenvolvedor, quero definir o `.gitignore` para excluir artefatos locais e de build.
- Como desenvolvedor, quero preparar scripts Docker para subir o ambiente (banco + aplicação) localmente.
- Como desenvolvedor, quero criar um arquivo de configuração de ambiente (`.env` ou similar) para variáveis sensíveis.
- Como desenvolvedor, quero definir pipelines de CI básico (build + testes) no GitHub Actions.
- Como desenvolvedor, quero documentar os passos de inicialização no `README` para novos contribuintes.
- Como desenvolvedor, quero escrever testes automatizados para as operações principais (NC, NE, consultas financeiras).
