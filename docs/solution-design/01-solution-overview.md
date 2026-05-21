# 01 — Visão Geral da Solução

> **Camada:** Foundation | **Depende de:** requirements/

## Problema

Organizações que precisam aprender ou demonstrar gestão orçamentária e financeira governamental não dispõem de um sistema acessível, moderno e funcional que reproduza os conceitos do SIAFI. O sistema real (SERPRO) não é open source, exige habilitação formal e roda em ambiente mainframe legado.

## Solução

O SIFU é uma aplicação web que implementa os principais fluxos do SIAFI em uma stack moderna: backend Java Spring Boot, frontend React, banco PostgreSQL. O sistema é funcional — não apenas didático — cobrindo o ciclo completo da despesa pública: dotação → NC → NE → liquidação → pagamento.

## Stakeholders

| Perfil | Interesse |
|---|---|
| Operador | Executar operações do dia a dia do sistema |
| Gestor Financeiro | Aprovar NCs e monitorar execução orçamentária |
| Ordenador de Despesas | Autorizar empenhos e consultar execução |
| Administrador | Manter usuários, hierarquia de UGs e auditoria |
| Sistema externo | Integrar via API com token de acesso |

## Escopo da Solução

**Incluído:**
- Autenticação com login/senha + tokens JWT para API
- CRUD de classificações orçamentárias e UGs
- Ciclo completo: Dotação → NC → NE → Liquidação → OB
- Consultas de execução orçamentária com filtros
- Dashboard de métricas financeiras
- API REST versionada com OpenAPI/Swagger
- Auditoria completa de todas as operações

**Excluído (Fase 1):**
- Integração bancária real
- Restos a Pagar (inscrição ao encerramento do exercício)
- Assinatura digital de documentos
- Emissão de PDF dos documentos
- BI avançado / relatórios complexos

## Decisões Arquiteturais Fundamentais

| Decisão | Escolha | Justificativa |
|---|---|---|
| Arquitetura | Monolito modular | Escopo educacional; complexidade de microsserviços não é justificada |
| Backend | Java 21 + Spring Boot 3 | Tecnologia do SIAFI real; ecossistema maduro para sistemas financeiros |
| Frontend | React + Vite | SPA adequada para sistema transacional interno; sem necessidade de SSR |
| Banco | PostgreSQL 16 | ACID nativo; suporte robusto a JSONB para auditoria; open source |
| API | REST + JSON | Padrão moderno de integração; melhor tooling que SOAP |
| Auth | JWT (sessão web) + API Token (integração) | Dois mecanismos com casos de uso distintos |

## Restrições

- Todas as operações financeiras devem ser ACID
- Nenhum documento financeiro pode ser excluído fisicamente
- Código-fonte em português
- Cobertura mínima: 80% geral, 90% nos fluxos críticos
- API versionada em `/api/v1`
