# 01 — Visão Geral

## Propósito

O **SIFU (Sistema Integrado Financeiro Unificado)** é um sistema educacional que implementa os conceitos de gestão orçamentária e financeira similares aos do SIAFI (Sistema Integrado de Administração Financeira) da SERPRO.

**Objetivos educacionais:**
- Demonstrar como sistemas financeiros governamentais complexos são estruturados
- Ensinar padrões de integração entre múltiplos órgãos e instituições
- Ilustrar práticas de segurança, auditoria e controle em sistemas críticos
- Servir como referência para desenvolvimento e integração com sistemas similares

## Objetivos do Sistema

- Fornecer gestão orçamentária e financeira para múltiplas instituições
- Permitir consultas e relatórios de execução orçamentária
- Gerenciar Notas de Crédito e Notas de Empenho
- Demonstrar integração entre múltiplos órgãos e unidades gestoras
- Garantir controle e auditoria completa de todas as operações financeiras

## Escopo

### No Escopo

- Consultar execução orçamentária
- Registrar e gerenciar Notas de Crédito entre órgãos
- Registrar e gerenciar Notas de Empenho
- Consultar empenhos por estado (emitidos, liquidados, pagos)
- Gerar dashboards financeiros
- Filtrar informações por classificações orçamentárias
- Gerenciar múltiplas instituições e órgãos em hierarquia

### Fora do Escopo

- Integração completa com sistemas bancários
- Gestão avançada de contratos
- BI avançado

## Usuários do Sistema

Todos os usuários têm **acesso ADMIN completo** — podem executar todas as operações. Esta simplificação é proposital para fins educacionais.

Perfis de uso sugeridos (para referência dos fluxos de negócio):

| Perfil | Descrição |
|---|---|
| Operador | Executa operações diárias do sistema |
| Gestor Financeiro | Responsável pela revisão financeira |
| Ordenador de Despesas | Responsável pela autorização de despesas |
| Administrador | Responsável pela manutenção, configuração e hierarquia de órgãos |

## Restrições

- Autenticação com login e senha é obrigatória para todos os acessos
- Todas as operações devem ser auditadas com registro completo
- Integrações entre órgãos requerem comunicação segura
- Implementação de segurança adequada (criptografia e autenticação forte)
- Disponibilidade esperada de 99,5%

## Fluxo de Execução da Despesa

```
1. Dotação criada
2. Nota de Crédito (opcional — transferência entre órgãos)
3. Nota de Empenho (reserva de orçamento)
4. Liquidação (confirmação de recebimento)
5. Pagamento via Ordem Bancária
```

## Possíveis Extensões Futuras

- Integração bancária
- Integração com sistemas de compras
- Controle de contratos
- Integração com sistemas logísticos
- BI avançado
