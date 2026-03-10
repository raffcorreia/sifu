# 01 — Visão Geral

## Propósito

Este documento descreve o **SIFU (Sistema Integrado Financeiro Unificado)**, um sistema educacional que implementa os conceitos de gestão orçamentária e financeira similares aos utilizados no SIAFI (Sistema Integrado de Administração Financeira) da SERPRO.

**Objetivo Educacional:**
- Demonstrar como sistemas financeiros governamentais complexos são estruturados
- Ensinar padrões de integração entre múltiplos órgãos e instituições
- Ilustrar práticas de segurança, auditoria e controle em sistemas críticos
- Servir como referência para desenvolvimento e integração com sistemas similares

Este documento descreve **como implementar um sistema financeiro integrado do zero**, utilizando conceitos reais de gestão orçamentária: consultas contábeis, **Nota de Crédito (NC)**, **Nota de Empenho (NE)** e dashboards de execução orçamentária.

## Objetivos

- Fornecer um sistema para gestão orçamentária e financeira multi-institucional
- Permitir consultas e relatórios de execução orçamentária
- Gerenciar Notas de Crédito e Empenho
- Demonstrar integração entre múltiplos órgãos e unidades gestoras
- Garantir controle e auditoria completa de operações financeiras

## Escopo

### No Escopo

- Consultar execução orçamentária
- Registrar e gerenciar Notas de Crédito entre órgãos
- Registrar e gerenciar Notas de Empenho
- Consultar empenhos (emitidos, liquidados, pagos)
- Gerar dashboards financeiros
- Filtrar informações por classificações orçamentárias
- Gerenciar múltiplas instituições e órgãos em uma hierarquia

### Fora do Escopo

- Integração completa com sistemas bancários
- Gestão de contratos avançada
- BI avançado
Usuários do Sistema

**Todos os usuários têm acesso ADMIN completo ao sistema** - podem executar todas as operações. Esta simplificação é proposital para fins educacionais.

Perfis de uso sugeridos (para referência de fluxos de negócio):

| Perfil | Descrição |
|---|---|
| Operador | Executa operações diárias do sistema |
| Gestor Financeiro | Responsável pela revisão financeira |
| Ordenador de Despesas | Responsável pela autorização de despesas |
| Administrador | Responsável pela manutenção e configuração
| Administrador | Mantepor múltiplos órgãos e instituições em uma hierarquia
- Autenticação com login e senha encriptada é obrigatória
- Todas as operações devem ser auditadas com registro completo
- Integrações entre órgãos requerem comunicação segura

## Restrições

- Implementação de segurança adequada (encriptação, autenticação forte)
- Disponibilidade esperada de 99.5%
- Suporte para integração entre múltiplos órgãos
## Restrições

- Deve seguir normas governamentais de segurança
- Disponibilidade de 99.5%

## Fluxo de Execução da Despesa

Fluxo principal:

1. Dotação criada
2. Nota de Crédito (opcional)
3. Nota de Empenho
4. Liquidação
5. Pagamento

## Possíveis Extensões Futuras

- Integração bancária
- Integração com sistemas de compras
- Controle de contratos
- Integração com sistemas logísticos
- BI avançado