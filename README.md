# SIFU — Sistema Integrado Financeiro Unificado

O **SIFU** é um sistema educacional de gestão orçamentária e financeira, inspirado no SIAFI (SERPRO). Seu objetivo é demonstrar como sistemas financeiros governamentais são estruturados e como outros sistemas podem se integrar a eles.

O sistema cobre o ciclo completo da execução da despesa pública: da criação de dotações orçamentárias até o pagamento ao fornecedor, passando por Notas de Crédito, Empenhos, Liquidações e Ordens Bancárias.

---

## Documentação de Requisitos

A documentação de requisitos está em `docs/requirements/`:

| Arquivo | Conteúdo |
|---|---|
| `01-overview.md` | Visão geral, objetivos e escopo |
| `02-data-model.md` | Modelo de dados e entidades principais |
| `03-functional-requirements.md` | Requisitos funcionais |
| `04-user-stories.md` | Histórias de usuário por perfil |
| `05-non-functional-requirements.md` | Segurança, qualidade, performance e arquitetura |

---

## Estrutura do Projeto

À medida que o sistema for implementado, cada funcionalidade receberá documentação incremental em `docs/`, organizada por história ou tema técnico. Cada pasta deve conter um `README.md` com:

1. Objetivo da funcionalidade
2. Como usar (telas, endpoints ou comandos)
3. Detalhes técnicos de configuração e execução
4. Procedimentos de teste e validação

---

## Como Contribuir

1. Crie uma branch com o nome da história (`feature/nota-empenho`, `tech/db-setup`).
2. Implemente o recurso.
3. Adicione ou atualize o `README.md` correspondente à história.
4. Abra um pull request com evidência de execução dos testes.
