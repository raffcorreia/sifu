# 03 — Requisitos Funcionais

## Contexto

O SIFU (Sistema Integrado Financeiro Unificado) é um sistema educacional que implementa funcionalidades de gestão orçamentária e financeira para múltiplos órgãos e instituições.

## Consultas Financeiras

**Consultas principais que o sistema deve suportar:**

- saldo orçamentário
- empenhos por período
- empenhos liquidados
- empenhos pagos
- execução por ação orçamentária
- execução por plano interno

**Filtros:**

- UG
- Exercício
- Ação Orçamentária
- Plano Interno
- Natureza Despesa
- Fonte
- PTRES
- Período

## 12. Dashboard

**Métricas:**

- Crédito disponível
- Total empenhado
- Total liquidado
- Total pago

## Requisitos Funcionais

- **RF01**: Autenticação via tela de login com usuário e senha para todos os acessos.
- **RF02**: Geração de tokens de acesso (JWT ou similar) para uso por sistemas externos na integração.
- **RF03**: Permitir cadastro e manutenção de classificações orçamentárias.
- **RF04**: Permitir criação e manutenção de dotações orçamentárias.
- **RF05**: Permitir criação de Nota de Crédito.
- **RF06**: Permitir aprovação ou cancelamento de Nota de Crédito.
- **RF07**: Permitir criação de Nota de Empenho.
- **RF08**: Validar saldo antes de emitir empenho.
- **RF09**: Permitir liquidação de empenho.
- **RF10**: Permitir registro de pagamento via Ordem Bancária.
- **RF11**: Permitir consultas de execução orçamentária.
- **RF12**: Permitir geração de dashboards financeiros.
- **RF13**: APIs expostas para integração com geração e revogação de tokens de acesso.
- **RF14**: Tela de login deve suportar logout, esqueci senha e validação de força de senha.
- **RF15**: Mesmo que o sistema categorize menus por papel (Operador, Gestor, etc.), todos os usuários têm privilégios administrativos completos.
- **RF16**: Permitir que usuários autenticados gerem, listem e revoguem tokens de integração para uso por sistemas externos.
- **RF17**: Todo token de integração deve estar vinculado ao usuário emissor e registrar data de criação, expiração e status.
- **RF18**: A aplicação deve expor API versionada para integração externa das operações previstas no escopo.
- **RF19**: A API deve possuir documentação funcional dos endpoints, parâmetros, payloads, códigos de resposta e exemplos de uso.