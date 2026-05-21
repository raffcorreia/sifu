# Plano de Implementação — SIFU

**Status**: Rascunho
**Criado**: 2026-05-20

---

## Índice

- [Resumo do Escopo](#resumo-do-escopo)
  - [O Que Está Sendo Construído](#o-que-está-sendo-construído)
  - [O Que Está Explicitamente Fora do Escopo](#o-que-está-explicitamente-fora-do-escopo)
  - [Requisitos Chave](#requisitos-chave)
  - [Stack Tecnológica](#stack-tecnológica)
  - [Resumo das Regras de Negócio Chave](#resumo-das-regras-de-negócio-chave)
- [Índice de Fases](#índice-de-fases)
- [Fases](#fases)
  - [FASE-000 — Scaffolding e Infraestrutura](#fase-000--scaffolding-e-infraestrutura)
  - [FASE-001 — Autenticação e Segurança](#fase-001--autenticação-e-segurança)
  - [FASE-002 — UGs e Classificações Orçamentárias](#fase-002--ugs-e-classificações-orçamentárias)
  - [FASE-003 — Dotações e Notas de Crédito](#fase-003--dotações-e-notas-de-crédito)
  - [FASE-004 — Fornecedores e Notas de Empenho](#fase-004--fornecedores-e-notas-de-empenho)
  - [FASE-005 — Liquidação e Ordem Bancária](#fase-005--liquidação-e-ordem-bancária)
  - [FASE-006 — Consultas, Dashboard e Tokens de Integração](#fase-006--consultas-dashboard-e-tokens-de-integração)
  - [FASE-007 — Observabilidade e Deploy](#fase-007--observabilidade-e-deploy)
  - [FASE-008 — Dados de Demonstração](#fase-008--dados-de-demonstração)
- [Fases de Correção de Bugs](#fases-de-correção-de-bugs)
- [Notas](#notas)

---

## Resumo do Escopo

O **SIFU (Sistema Integrado Financeiro Unificado)** é um sistema web educacional que reproduz os principais fluxos do SIAFI — o sistema financeiro do Governo Federal brasileiro. É funcional, não apenas didático, cobrindo o ciclo completo da despesa pública.

### O Que Está Sendo Construído

- Backend monolítico modular Java Spring Boot 3 + PostgreSQL 16
- Frontend SPA React inspirado visualmente no SIAFI Web
- Ciclo completo: Dotação → Nota de Crédito → Nota de Empenho → Liquidação → Ordem Bancária
- Autenticação por login/senha (JWT 8h) e tokens de integração para API externa
- CRUD de UGs, hierarquia organizacional e classificações orçamentárias
- Consultas de execução orçamentária e dashboard financeiro
- Auditoria completa de todas as operações financeiras
- CI/CD via GitHub Actions

### O Que Está Explicitamente Fora do Escopo

- Integração bancária real (OB é simulação)
- Restos a Pagar (inscrição ao encerramento do exercício)
- Assinatura digital de documentos
- Emissão de PDF
- BI avançado / relatórios complexos
- Controle de acesso por perfil (todos os usuários têm acesso ADMIN)
- Entidade `Gestao` (simplificação educacional — ver 00-index.md)
- Entidade `PerfilAcesso` (todos os usuários são ADMIN)

### Requisitos Chave

| ID | Requisito | Descrição |
|---|---|---|
| RF01 | Login | Autenticação com login/senha, bloqueio após 5 tentativas inválidas |
| RF02 | Tokens JWT | Sessões JWT de 8h para usuários web |
| RF03 | Classificações Orçamentárias | CRUD de UG, Ação Orçamentária, Plano Interno, ND, FR, PTRES |
| RF04 | Dotações | Criar, suplementar e consultar saldo de dotações orçamentárias |
| RF05/06 | Nota de Crédito | Emitir, aprovar, cancelar e estornar NCs entre UGs |
| RF07/08 | Nota de Empenho | Emitir, reforçar, anular NEs; cadastrar fornecedores |
| RF09 | Liquidação | Registrar e estornar liquidações de empenho |
| RF10 | Ordem Bancária | Emitir, processar e cancelar OBs |
| RF11/12 | Consultas / Dashboard | Execução orçamentária com filtros; métricas financeiras |
| RF13–17 | Tokens de Integração | Gerar, listar, revogar tokens de API |
| RF18/19 | API + Docs | API REST versionada em `/api/v1` com Swagger |
| RNF-SEG | Segurança | BCrypt-12, brute force, JWT, CORS, headers de segurança |
| RNF-AUD | Auditoria | Registro completo de todas as operações via AOP |
| RNF-PERF | Performance | Listagens < 2s; consultas orçamentárias < 3s |
| RNF-QUAL | Qualidade | Cobertura ≥ 80% geral, ≥ 90% fluxos críticos |
| RNF-DEP | Deploy | Docker Compose; CI/CD GitHub Actions |

### Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Linguagem Backend | Java 21 (LTS) |
| Framework Backend | Spring Boot 3.3.x |
| ORM | Hibernate 6 via Spring Data JPA |
| Migrações | Flyway 10.x |
| Mapeamento | MapStruct 1.5.x |
| Segurança | Spring Security + JJWT 0.12.x |
| Documentação API | SpringDoc OpenAPI 2.x (Swagger UI) |
| Testes Backend | JUnit 5 + Mockito + Testcontainers (PostgreSQL real) |
| Build Backend | Maven 3.9+ |
| Framework Frontend | React 18 + Vite 5 |
| Roteamento | React Router 6 |
| HTTP Client | Axios 1.x |
| Formulários | React Hook Form 7 + Zod 3.x |
| Tabelas | TanStack Table 8.x |
| Testes Frontend | Jest 29.x + React Testing Library 14.x |
| Banco de Dados | PostgreSQL 16 |
| Containerização | Docker + Docker Compose |
| CI/CD | GitHub Actions |

### Resumo das Regras de Negócio Chave

| Regra | Descrição |
|---|---|
| RN-01 | Documentos financeiros não são editáveis após emissão |
| RN-02 | Exclusão física nunca ocorre — status `INACTIVE`/`VOIDED`/`CANCELLED` |
| RN-03 | Numeração automática: `[exercício 4d][código UG sem traços][sequencial 6d]` (SELECT FOR UPDATE) |
| RN-04 | Saldo = dotação_atualizada + NCs_aprovadas_recebidas − NCs_aprovadas_cedidas − NEs_ativas |
| RN-05 | NE ORDINARY: anulação somente total; ESTIMATED/GLOBAL: anulação parcial permitida |
| RN-06 | Não é possível estornar NL com OB vinculada; não é possível anular NE com NL vigente |
| RN-07 | NC deve ter UG origem ≠ UG destino |
| RN-08 | Brute force: 5 tentativas inválidas → bloqueio de 15 minutos |
| RN-09 | Token de integração exibido apenas na criação; armazenado como hash SHA-256 |
| RN-10 | Todas as operações financeiras são ACID (transações Spring `@Transactional`) |

---

## Índice de Fases

| ID da Fase | Nome | Mapeamento FR/QA | Dependências |
|---|---|---|---|
| FASE-000 | Scaffolding e Infraestrutura | — (fundação) | Nenhuma |
| FASE-001 | Autenticação e Segurança | RF01, RF02 — QA FASE 1 | FASE-000 |
| FASE-002 | UGs e Classificações Orçamentárias | RF03 — QA FASE 2 | FASE-001 |
| FASE-003 | Dotações e Notas de Crédito | RF04, RF05, RF06 — QA FASE 3 | FASE-002 |
| FASE-004 | Fornecedores e Notas de Empenho | RF07, RF08 — QA FASE 4 | FASE-003 |
| FASE-005 | Liquidação e Ordem Bancária | RF09, RF10 — QA FASE 5 | FASE-004 |
| FASE-006 | Consultas, Dashboard e Tokens de Integração | RF11–RF19 — QA FASE 6 | FASE-005 |
| FASE-007 | Observabilidade e Deploy | — (operacional) | FASE-006 |
| FASE-008 | Dados de Demonstração | — (dados) | FASE-007 |

---

## Fases

---

### FASE-000 — Scaffolding e Infraestrutura

#### Identidade da Fase

**ID da Fase**: FASE-000
**Nome da Fase**: Scaffolding e Infraestrutura
**Mapeamento FR**: Nenhum (fundação)

#### Objetivo

O projeto está estruturado com backend Spring Boot e frontend React funcionando localmente via Docker Compose. O PostgreSQL 16 sobe saudável, Flyway está configurado, e as camadas base do backend (pacotes, exceções comuns, Spring Security stub) e do frontend (roteamento, contextos, páginas vazias) existem e compilam sem erros.

#### Dependências

Nenhuma

#### Pré-requisitos

- Docker Desktop instalado e em execução
- Java 21 JDK instalado localmente
- Maven 3.9+ instalado localmente
- Node.js 22 instalado localmente

#### Pacotes Backend (pom.xml)

| Dependência | Escopo | Propósito |
|---|---|---|
| `spring-boot-starter-web` | compile | REST controllers |
| `spring-boot-starter-data-jpa` | compile | JPA / Hibernate |
| `spring-boot-starter-security` | compile | Spring Security |
| `spring-boot-starter-validation` | compile | Bean Validation (`@Valid`) |
| `spring-boot-starter-mail` | compile | Recuperação de senha (SMTP) |
| `spring-boot-starter-actuator` | compile | Health check e métricas |
| `spring-boot-starter-aop` | compile | AOP para auditoria |
| `spring-boot-starter-test` | test | JUnit 5 + Mockito |
| `flyway-core` | compile | Migrações SQL |
| `postgresql` | runtime | Driver JDBC |
| `lombok` | compile | Geração de boilerplate |
| `mapstruct` | compile | Mapeamento DTO ↔ entidade |
| `mapstruct-processor` | provided | Processador de anotações MapStruct |
| `springdoc-openapi-starter-webmvc-ui` | compile | Swagger UI / OpenAPI |
| `jjwt-api` | compile | JWT API |
| `jjwt-impl` | runtime | JWT implementation |
| `jjwt-jackson` | runtime | JWT JSON parsing |
| `testcontainers` (BOM) | test | Testcontainers BOM |
| `testcontainers-postgresql` | test | PostgreSQL real para testes |
| `testcontainers-junit-jupiter` | test | Integração JUnit 5 |

#### Pacotes Frontend (package.json)

| Pacote | Tipo | Propósito |
|---|---|---|
| `react`, `react-dom` | prod | UI library |
| `react-router-dom` | prod | Roteamento SPA |
| `axios` | prod | HTTP client |
| `react-hook-form` | prod | Gerenciamento de formulários |
| `@hookform/resolvers` | prod | Integração Zod + RHF |
| `zod` | prod | Validação de schema |
| `@tanstack/react-table` | prod | Tabelas com paginação/filtro |
| `date-fns` | prod | Formatação de datas e valores |
| `vite`, `@vitejs/plugin-react` | dev | Build tool |
| `jest`, `jest-environment-jsdom` | dev | Test runner |
| `@testing-library/react` | dev | Component testing |
| `@testing-library/user-event` | dev | Simulação de interações |
| `@testing-library/jest-dom` | dev | Matchers DOM |
| `@babel/preset-env`, `@babel/preset-react` | dev | Transpilação JS/JSX |
| `eslint`, `eslint-plugin-react` | dev | Linting |

#### Plano

1. Criar a estrutura de diretórios do repositório conforme `12-deployment.md`:
   - `sifu/backend/`
   - `sifu/frontend/`
2. Criar `.env.example` na raiz com os oito campos documentados em `12-deployment.md`: `DB_PASSWORD`, `JWT_SECRET`, `CORS_ALLOWED_ORIGINS`, `EMAIL_HOST`, `EMAIL_PORTA`, `EMAIL_USUARIO`, `EMAIL_PASSWORD`, `RATE_LIMIT_RPM` — todos com valores placeholder (sem credenciais reais)
3. Criar `.env` na raiz (gitignored) com valores de desenvolvimento local
4. Criar `docker-compose.yml` na raiz conforme o design em `12-deployment.md` com os três serviços: `banco` (PostgreSQL 16), `backend`, `frontend`
5. Inicializar o projeto Maven em `backend/`:
   - `groupId: br.gov.sifu`, `artifactId: sifu-backend`, Java 21, encoding UTF-8
   - Adicionar todas as dependências listadas na seção "Pacotes Backend"
   - Configurar `maven-compiler-plugin` com `-Amapstruct.defaultComponentModel=spring` e `-Amapstruct.unmappedTargetPolicy=ERROR`
   - Configurar `jacoco-maven-plugin` com threshold de cobertura geral 80% e pacotes críticos 90% (`auth`, `commitment`, `settlement`, `paymentorder`, `creditnote`)
6. Criar a estrutura completa de pacotes em `backend/src/main/java/br/gov/sifu/` conforme `05-backend-design.md`:
   - `auth/` (com subpacote `dto/`)
   - `managingunit/` (com `dto/`)
   - `classification/budgetaction/`, `classification/internalplan/`, `classification/expensenature/`, `classification/fundingsource/`, `classification/ptres/`
   - `allotment/`, `creditnote/`, `vendor/`, `commitment/`, `settlement/`, `paymentorder/`
   - `report/`, `user/`, `token/`, `audit/`
   - `common/exception/`, `common/pagination/`, `common/security/`, `common/sequence/`
7. Criar as classes base em `common/`:
   - `ProblemDetail.java` — DTO de erro RFC 7807 com campos: `tipo`, `titulo`, `status`, `detalhe`, `instancia`
   - `GlobalExceptionHandler.java` — `@ControllerAdvice` com estrutura RFC 7807 (handlers serão adicionados por fase)
   - `PaginationUtils.java` — método `construirPageable(Integer pagina, Integer tamanho, String ordenarPor, String direcao)` com validação de `tamanho` máximo 100
   - `SecurityConfig.java` — `@Configuration @EnableWebSecurity` com `permitAll()` em todos os endpoints (stub provisório; substituído na FASE-001)
8. Criar `backend/src/main/resources/application.yml` com:
   - `spring.datasource.url/username/password` usando variáveis de ambiente (`${SPRING_DATASOURCE_URL}` etc.)
   - `spring.jpa.hibernate.ddl-auto: validate`
   - `spring.flyway.enabled: true` / `spring.flyway.locations: classpath:db/migration`
   - `spring.mail.*` configuração SMTP via variáveis de ambiente
   - `springdoc.swagger-ui.path: /swagger-ui.html`
   - `management.endpoints.web.exposure.include: health`
9. Criar diretório vazio `backend/src/main/resources/db/migration/` (migrações adicionadas por fase)
10. Criar `SifuApplication.java` com `@SpringBootApplication` no pacote raiz `br.gov.sifu`
11. Verificar que o backend compila sem erros: `mvn compile -f backend/pom.xml`
12. Inicializar o projeto Vite + React em `frontend/`:
    - `npm create vite@latest . -- --template react` dentro de `frontend/`
    - Instalar todos os pacotes listados na seção "Pacotes Frontend"
13. Criar a estrutura de diretórios do frontend em `frontend/src/`:
    - `components/common/` — componentes reutilizáveis (`SystemHeader`, `SideMenu`, `StatusBadge`, `ConfirmationModal`, `TabelaDados`, `SearchSelector`)
    - `pages/` — subdiretório por módulo (a popular nas fases seguintes)
    - `services/` — clientes HTTP via Axios (um arquivo por módulo)
    - `contexts/` — React Context (autenticação)
    - `hooks/` — custom hooks reutilizáveis
    - `utils/` — helpers de formatação (moeda BRL, datas ISO → BR)
14. Criar `frontend/src/main.jsx` com `RouterProvider` do React Router 6
15. Criar `frontend/src/router.jsx` com rotas stub para todos os módulos (cada rota retorna `<div>Em construção</div>` até ser implementada)
16. Criar `frontend/jest.config.js` com `testEnvironment: 'jsdom'` e `setupFilesAfterFramework` apontando para arquivo que importa `@testing-library/jest-dom`
17. Criar `frontend/Dockerfile` e `frontend/nginx.conf` conforme `12-deployment.md`
18. Criar `backend/Dockerfile` conforme `12-deployment.md`
19. Criar `.github/workflows/ci.yml` com dois jobs stub que fazem apenas `checkout` e retornam sucesso (completado na FASE-007)
20. Verificar: `docker compose up -d banco` → `pg_isready` retorna saudável; `mvn compile` passa; `npm ci` instala sem erros

#### Resultado Esperado

- Estrutura de repositório completa (`backend/`, `frontend/`, `docker-compose.yml`, `.env.example`, `.github/`)
- Backend Maven compila sem erros; Spring Boot inicia e conecta ao banco via Docker Compose
- Frontend React inicia em modo de desenvolvimento sem erros
- PostgreSQL 16 saudável no Docker
- Flyway configurado aguardando migrações
- `.env.example` commitado com placeholders; `.env` gitignored
- Dockerfiles para backend e frontend criados

#### Verificação Automatizada (AI Gate)

1. `mvn compile -f backend/pom.xml` — compilação sem erros
2. `docker compose up -d banco` seguido de `docker exec sifu-banco pg_isready -U sifu` — retorna "accepting connections"
3. `npm ci --prefix frontend` — dependências instaladas sem erros
4. `npx jest --passWithNoTests --prefix frontend` — runner Jest inicia sem erros de configuração
5. Verificar que `.env` está no `.gitignore` e `git status` não o exibe

**Esperado:**
- Backend compila; PostgreSQL acessível em `localhost:5432`; Jest configurado

#### Verificação Humana (Human Gate)

1. Docker Desktop exibe container `sifu-banco` saudável
2. Revisar estrutura de pacotes do backend — confirmar que reflete `05-backend-design.md`
3. Confirmar que `.env` não aparece no `git status`
4. Confirmar que `.env.example` contém apenas placeholders

#### Critérios de Sucesso

✅ `backend/`, `frontend/`, `docker-compose.yml`, `.env.example`, `.github/` criados e commitados
✅ `mvn compile` passa sem erros
✅ Spring Boot inicia e conecta ao PostgreSQL via `docker compose up`
✅ `npm ci` instala sem erros
✅ Jest configurado com `jsdom` e `@testing-library/jest-dom`
✅ `.env` gitignored; `.env.example` commitado com placeholders
✅ Dockerfiles backend e frontend criados
✅ Estrutura de pacotes backend conforme `05-backend-design.md`

---

### FASE-001 — Autenticação e Segurança

#### Identidade da Fase

**ID da Fase**: FASE-001
**Nome da Fase**: Autenticação e Segurança
**Mapeamento FR**: RF01, RF02 | **QA**: FASE 1 (`13-qa-process.md`)

#### Objetivo

O sistema de autenticação está completamente funcional: login com JWT, logout, recuperação de senha por e-mail, redefinição de senha, bloqueio por tentativas inválidas, Swagger acessível sem autenticação. O usuário admin padrão é criado pela migration V8. O frontend tem a página de login integrada e rotas protegidas funcionando.

#### Dependências

FASE-000 Scaffolding e Infraestrutura

#### Plano

1. Criar migração `V1__create_security_tables.sql` em `backend/src/main/resources/db/migration/`:
   - Tabelas `usuarios`, `tokens_integracao`, `tokens_redefinicao_senha` conforme DDL em `04-database-design.md`
2. Criar migração `V8__initial_data.sql`:
   - Inserir usuário `admin` com `login='admin'`, senha `Admin@123` hashada com BCrypt-12 e `status='ACTIVE'`
   - Inserir UG raiz de exemplo: `codigo_ug='MIN_ED'`, `nome='Ministério da Educação'`
3. Criar a entidade JPA `User` em `auth/` com todos os campos da tabela `usuarios`; implementar `UserDetails` do Spring Security
4. Criar `UserRepository` (JpaRepository) com: `findByLogin(String login)`, `findByEmail(String email)`
5. Implementar `JwtService` em `auth/`:
   - `gerarToken(User user)` — JWT com claims: `sub=login`, `nome`, `tipo=SESSAO`, `exp` = agora + 8h; assinar com HMAC-SHA256 usando `JWT_SECRET`
   - `extrairLogin(String token)` — extrai o subject
   - `validarToken(String token)` — verifica assinatura e expiração; lança `InvalidTokenException` ou `ExpiredTokenException`
6. Implementar `JwtFilter` (`OncePerRequestFilter`):
   - Extrai token do header `Authorization: Bearer <token>`
   - Autentica via `JwtService`; popula `SecurityContextHolder`
   - Ignora endpoints públicos
7. Substituir `SecurityConfig` stub pela configuração real:
   - Endpoints públicos: `POST /api/v1/auth/login`, `POST /api/v1/auth/recover-password`, `GET /swagger-ui/**`, `GET /api-docs/**`, `GET /actuator/health`
   - Todos os demais `authenticated()`
   - Adicionar `JwtFilter` antes de `UsernamePasswordAuthenticationFilter`
   - Configurar CORS com a origin `${CORS_ALLOWED_ORIGINS}`
   - Desabilitar CSRF (SPA com JWT); session stateless
8. Implementar `AuthService`:
   - `login(RequisicaoLogin req)` — verifica senha BCrypt; se inválida: incrementa `tentativasLogin`; se >= 5: seta `bloqueadoAte = agora + 15min`; se válida: zera `tentativasLogin` e retorna JWT via `JwtService`
   - `logout(String token)` — invalida sessão atual (simplificação: stateless, basta o cliente descartar o token)
   - `recuperarSenha(String email)` — busca usuário por e-mail; gera UUID, persiste em `tokens_redefinicao_senha` com `expira_em = agora + 1h`; envia e-mail via `JavaMailSender`; sempre retorna 204 (não revela existência do e-mail)
   - `redefinirSenha(RequisicaoRedefinicaoSenha req)` — busca token em `tokens_redefinicao_senha`; valida `usado=false` e `expira_em > agora`; atualiza `senha_hash` com BCrypt-12; marca token como `usado=true`
   - `alterarSenha(RequisicaoAlterarSenha req)` — verifica `senhaAtual` com BCrypt; atualiza para `novaSenha`
9. Implementar `AuthController` com os cinco endpoints de `06-api-design.md`: `POST /login`, `POST /logout`, `POST /recover-password`, `PUT /reset-password`, `PUT /change-password`
10. Criar `SecurityContext` em `common/security/` com método estático `getUsuarioLogado()` que extrai o `User` do `SecurityContextHolder`
11. Criar exceções em `common/exception/` com handlers no `GlobalExceptionHandler`:
    - `InvalidCredentialsException` → 401 RFC 7807
    - `AccountLockedException` → 423 RFC 7807 com detalhe do horário de desbloqueio
    - `InvalidTokenException` → 401 RFC 7807
    - `ExpiredTokenException` → 401 RFC 7807
    - `EntityNotFoundException` → 404 RFC 7807
    - Handler para `MethodArgumentNotValidException` (Bean Validation) → 400 RFC 7807 com lista de campos inválidos
12. Escrever testes unitários em `AuthServiceTest` (pacote `auth`):
    ```
    // Constantes obrigatórias no topo da classe:
    static final String VALID_LOGIN         = "joao.silva";
    static final String VALID_PASSWORD      = "MinhaS3nha!";
    static final String INVALID_PASSWORD    = "SenhaErrada";
    static final String VALID_EMAIL         = "joao@orgao.gov.br";
    static final String USER_NAME           = "João da Silva";
    static final int    MAX_ATTEMPTS        = 5;
    static final int    LOCKOUT_MINUTES     = 15;
    ```
    - `shouldReturnTokenWhenCredentialsValid()`
    - `shouldThrowExceptionWhenPasswordIncorrect()`
    - `shouldIncrementLoginAttemptsWhenPasswordIncorrect()`
    - `shouldLockAccountAfter5InvalidAttempts()`
    - `shouldThrowAccountLockedWhenAccountIsLocked()`
    - `shouldGeneratePasswordResetTokenWhenEmailValid()`
    - `shouldResetPasswordWhenTokenValid()`
    - `shouldThrowExceptionWhenResetTokenExpired()`
    - `shouldThrowExceptionWhenResetTokenAlreadyUsed()`
13. Escrever testes de integração com Testcontainers em `AuthIntegrationTest`:
    - Constantes: `ADMIN_LOGIN`, `ADMIN_PASSWORD`, `WRONG_PASSWORD`, `MAX_ATTEMPTS`, `LOGIN_URL`
    - Cobrir todos os casos do QA FASE 1 (F1-01 a F1-10 de `13-qa-process.md`)
    - Verificar que migration V8 cria o usuário admin
    - Verificar `POST /api/v1/auth/login` retorna 200 com token JWT para `admin/Admin@123`
    - Verificar `POST /api/v1/auth/login` retorna 401 para senha incorreta
    - Verificar bloqueio após 5 tentativas retorna 423
    - Verificar requisição sem token a endpoint protegido retorna 401
    - Verificar Swagger UI acessível sem token (`GET /swagger-ui.html` → 200)
14. Criar `frontend/src/services/authService.js`:
    - `login(credenciais)` — `POST /api/v1/auth/login`
    - `logout()` — `POST /api/v1/auth/logout`
    - `recuperarSenha(email)` — `POST /api/v1/auth/recover-password`
    - `redefinirSenha(dados)` — `PUT /api/v1/auth/reset-password`
15. Criar `frontend/src/contexts/AuthContext.jsx`:
    - Estado: `usuario`, `token`, `autenticado`
    - Funções: `entrar(credenciais)`, `sair()`
    - Persistência do token em `localStorage`
    - Interceptor Axios global que adiciona `Authorization: Bearer <token>` a todas as requisições
    - Interceptor de resposta que chama `sair()` e redireciona para `/login` em 401
16. Criar `frontend/src/pages/Login/LoginPage.jsx`:
    - Formulário com campos `login` e `senha` validados com React Hook Form + Zod
    - Exibe "Credenciais inválidas" em 401
    - Exibe "Conta bloqueada. Tente novamente em X minutos." em 423
    - Redireciona para `/` após login bem-sucedido
    - Botão desabilitado com indicador de loading durante a requisição
17. Criar `frontend/src/components/ProtectedRoute.jsx` — wrapper que redireciona para `/login` se `!autenticado`
18. Atualizar `router.jsx` para envolver todas as rotas de módulos em `<ProtectedRoute />`
19. Escrever testes de componente em `LoginPage.test.jsx`:
    ```js
    // Constantes obrigatórias no topo:
    const VALID_LOGIN       = 'admin';
    const VALID_PASSWORD    = 'Admin@123';
    const ERRO_CREDENCIAIS  = 'Credenciais inválidas';
    const ERRO_BLOQUEIO     = 'Conta bloqueada';
    ```
    - `shouldShowErrorWhenCredentialsInvalid()`
    - `shouldRedirectToHomeAfterSuccessfulLogin()`
    - `shouldDisableButtonWhileRequestInProgress()`
    - `shouldShowLockErrorWhenAccount423()`
20. Executar `mvn verify -f backend/pom.xml` — cobertura do pacote `auth` ≥ 90%

#### Resultado Esperado

- Migrations V1__create_security_tables e V8__initial_data aplicadas — tabelas de segurança e usuário admin criados
- Login retorna JWT válido; logout é funcional
- Recuperação de senha gera token (testado com mock SMTP); redefinição funciona
- Bloqueio após 5 tentativas inválidas por 15 minutos
- Todos os endpoints protegidos retornam 401 sem token
- Swagger UI acessível sem autenticação
- Frontend com página de login e rotas protegidas funcionais

#### Verificação Automatizada (AI Gate)

1. `mvn verify -f backend/pom.xml` — todos os testes passam; cobertura `auth` ≥ 90%
2. `POST /api/v1/auth/login` com `{"login":"admin","senha":"Admin@123"}` → 200 com `token` JWT
3. `POST /api/v1/auth/login` com senha incorreta → 401 RFC 7807
4. Após 5 tentativas inválidas → 423 RFC 7807
5. `GET /api/v1/managing-units` sem token → 401
6. `GET /swagger-ui.html` sem token → 200
7. `npm test --prefix frontend` — testes de `LoginPage` passam

**Esperado:**
- Fluxo de autenticação completo funcionando; endpoints protegidos retornam 401 corretamente

#### Verificação Humana (Human Gate)

1. Navegar para `http://localhost:5173` — redireciona para `/login`
2. Login com `admin` / `Admin@123` — redireciona para tela principal
3. Swagger UI em `http://localhost:8080/swagger-ui.html` acessível sem token
4. **QA executa os casos F1-01 a F1-10 (`13-qa-process.md`) e assina o aceite da FASE 1**

#### Critérios de Sucesso

✅ Migration V1__create_security_tables cria `usuarios`, `tokens_integracao`, `tokens_redefinicao_senha`
✅ Migration V8__initial_data cria usuário admin com senha BCrypt-12
✅ `POST /api/v1/auth/login` retorna JWT em 200
✅ Senha errada → 401 RFC 7807
✅ Conta bloqueada após 5 tentativas → 423
✅ Logout funcional
✅ Recuperação e redefinição de senha funcionam (mock SMTP em testes)
✅ Redefinição com token expirado → 401
✅ Swagger UI acessível sem autenticação
✅ Requisição sem token → 401
✅ Cobertura `auth` ≥ 90%
✅ Testes de integração cobrem F1-01 a F1-10
✅ Página de Login do frontend funcional
✅ Rotas protegidas redirecionam para `/login`
✅ **QA sign-off obtido (FASE 1)**

---

### FASE-002 — UGs e Classificações Orçamentárias

#### Identidade da Fase

**ID da Fase**: FASE-002
**Nome da Fase**: UGs e Classificações Orçamentárias
**Mapeamento FR**: RF03 | **QA**: FASE 2 (`13-qa-process.md`)

#### Objetivo

É possível cadastrar e manter a hierarquia de Unidades Gestoras e todas as cinco classificações orçamentárias (Ação Orçamentária, Plano Interno, Natureza de Despesa, Fonte de Recurso, PTRES) com CRUD completo via API e interface frontend.

#### Dependências

FASE-001 Autenticação e Segurança

#### Plano

1. Criar migração `V2__create_structure_tables.sql`:
   - Tabela `unidades_gestoras` com auto-referência `orgao_superior_id` conforme DDL em `04-database-design.md`
2. Criar migração `V3__create_classification_tables.sql`:
   - Tabelas `acoes_orcamentarias`, `planos_internos`, `naturezas_despesa`, `fontes_recurso`, `ptres` conforme DDL
   - Índice `idx_ug_codigo`, `idx_ug_superior`
3. Criar entidade JPA `ManagingUnit` em `managingunit/` com `@ManyToOne @JoinColumn(name="orgao_superior_id") ManagingUnit orgaoSuperior` (nullable)
4. Criar `ManagingUnitRepository`:
   - `findByCodigoUg(String codigo)`
   - `findByOrgaoSuperiorId(Long id)`
   - `findAllByStatus(String status, Pageable p)`
   - `existsByCodigoUg(String codigo)`
5. Criar DTOs em `managingunit/dto/`: `RequisicaoCriarUG`, `RequisicaoAtualizarUG`, `RespostaUG` (inclui `orgaoSuperior` simplificado: apenas `id` e `nome`)
6. Criar `ManagingUnitMapper` (MapStruct)
7. Implementar `ManagingUnitService`:
   - `create(req)` — valida código único
   - `findById(id)` — lança `EntityNotFoundException` se não encontrada
   - `list(filtros, pageable)`
   - `update(id, req)` — não permite alteração de `codigoUg`
   - `deactivate(id)` — status → `INACTIVE`
   - `findSubordinates(id)` — busca recursiva de subordinadas
8. Criar `ManagingUnitController` com endpoints: `GET /api/v1/managing-units`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/deactivate`, `GET /{id}/subordinates`
9. Para cada classificação (`BudgetAction`, `InternalPlan`, `ExpenseNature`, `FundingSource`, `Ptres`), criar entidade JPA, Repository, DTOs, Mapper e Service com operações: `create`, `findById`, `list`, `update`, `deactivate`, `delete`
10. A operação `excluir` em classificações deve verificar se há dotações referenciando a classificação; se sim → lançar nova exceção `ReferencedEntityException` → 422 RFC 7807 com mensagem "Classificação referenciada; não pode ser excluída"
11. Adicionar handler de `ReferencedEntityException` ao `GlobalExceptionHandler`
12. Criar Controllers para cada classificação com padrão CRUD + `PATCH /{id}/deactivate` + `DELETE /{id}` conforme `06-api-design.md`
13. Adicionar `@Auditable` stub (anotação criada nesta fase) aos métodos de escrita dos Services — o interceptor AOP só é ativado na FASE-006; por ora a anotação existe mas não faz nada
14. Escrever testes unitários `ManagingUnitServiceTest`:
    ```
    static final String UNIT_CODE          = "MIN_ED";
    static final String UNIT_NAME          = "Ministério da Educação";
    static final String DUPLICATE_UNIT_CODE = "MIN_ED"; // mesmo código para teste de duplicação
    static final Long   PARENT_UNIT_ID     = 1L;
    ```
    - `shouldCreateManagingUnitWhenDataValid()`
    - `shouldThrowExceptionWhenUnitCodeDuplicated()`
    - `shouldDeactivateExistingManagingUnit()`
    - `shouldThrowExceptionWhenManagingUnitNotFound()`
    - `shouldReturnSubordinatesOfManagingUnit()`
    - `shouldNotAllowUnitCodeChange()`
15. Escrever testes unitários análogos para pelo menos `BudgetActionServiceTest` e `InternalPlanServiceTest`; os demais seguem o mesmo padrão
16. Escrever testes de integração (Testcontainers) cobrindo F2-01 a F2-10 de `13-qa-process.md`
17. Criar componente reutilizável `SearchSelector` em `frontend/src/components/common/`:
    - Input de busca com ícone de lupa
    - Abre modal/dropdown com resultado paginado
    - Emite o item selecionado via `onChange`
    - Usado em formulários de dotação, NC, NE, etc.
18. Criar `frontend/src/pages/ManagingUnit/` com `ManagingUnitList.jsx` e `ManagingUnitForm.jsx`
19. Criar `frontend/src/pages/Classification/` com `ClassificationList.jsx` e `ClassificationForm.jsx` (componentes genéricos parametrizados por tipo)
20. Criar serviços HTTP: `managingUnitService.js`, `classificationService.js`
21. Escrever testes de componente:
    ```js
    // ManagingUnitList.test.jsx
    const MANAGING_UNITS_MOCK = [{ id: 1, codigoUg: 'MIN_ED', nome: 'Ministério da Educação', status: 'ACTIVE' }];
    const EMPTY_LIST_MESSAGE = 'Nenhuma unidade gestora encontrada';
    ```
    - `shouldShowLoadedManagingUnitList()`
    - `shouldShowMessageWhenListIsEmpty()`
    - `shouldShowValidationErrorWhenRequiredFieldEmpty()`
22. Executar `mvn verify` — cobertura geral ≥ 80%

#### Resultado Esperado

- Migrations V2 e V3 aplicadas
- CRUD de UG com hierarquia; UG desativada não aparece em novos cadastros
- CRUD completo das 5 classificações com desativação e exclusão (bloqueada se referenciada)
- Frontend com listagem e formulário para UG e classificações

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura geral ≥ 80%
2. `POST /api/v1/managing-units` → 201 com `id` gerado
3. `POST /api/v1/managing-units` com `codigoUg` duplicado → 422 RFC 7807
4. `GET /api/v1/managing-units/{id}/subordinates` → lista de subordinadas
5. `PATCH /api/v1/managing-units/{id}/deactivate` → 204
6. `DELETE /api/v1/budget-actions/{id}` em uso → 422; sem uso → 204
7. `npm test --prefix frontend` — testes de listagem e formulário passam

#### Verificação Humana (Human Gate)

1. Cadastrar UG raiz; cadastrar UG subordinada e verificar hierarquia
2. CRUD completo de uma classificação; tentar excluir classificação em uso — mensagem de erro correta
3. **QA executa os casos F2-01 a F2-10 (`13-qa-process.md`) e assina o aceite da FASE 2**

#### Critérios de Sucesso

✅ Migrations V2 e V3 aplicadas
✅ CRUD de UG com hierarquia (`orgaoSuperior`)
✅ UG desativada não aparece em novos cadastros
✅ CRUD completo de BudgetAction, InternalPlan, ExpenseNature, FundingSource, PTRES
✅ Exclusão de classificação referenciada → 422
✅ Testes unitários e de integração passam
✅ `SearchSelector` implementado e testado
✅ Frontend: listagem e formulário para UG e classificações
✅ **QA sign-off obtido (FASE 2)**

---

### FASE-003 — Dotações e Notas de Crédito

#### Identidade da Fase

**ID da Fase**: FASE-003
**Nome da Fase**: Dotações e Notas de Crédito
**Mapeamento FR**: RF04, RF05, RF06 | **QA**: FASE 3 (`13-qa-process.md`)

#### Objetivo

É possível criar dotações orçamentárias, calcular saldos e transferir créditos entre UGs via Notas de Crédito. A máquina de estados PENDING → APPROVED → REVERSED (e o caminho CANCELLED) está completamente funcional com validações de saldo. A numeração automática de documentos funciona atomicamente.

#### Dependências

FASE-002 UGs e Classificações Orçamentárias

#### Plano

1. Criar migração `V4__create_budget_tables.sql`:
   - Tabela `sequencias_documentos` (necessária desde esta fase para numeração de NCs)
   - Tabela `dotacoes_orcamentarias` com todos os FKs para classificações
   - Tabela `notas_credito`
   - Índice `idx_dotacao_ug_exercicio`, `idx_nc_numero`, `idx_nc_status`

   > **Nota de ajuste:** O design original (`04-database-design.md`) coloca `sequencias_documentos` em V5. Esta fase a move para V4 pois a numeração de NC já é necessária aqui. Os documentos de design não precisam ser atualizados — é uma decisão de implementação.

2. Implementar `DocumentNumberGenerator` em `common/sequence/`:
   - `gerarNumero(String tipoDocumento, Long ugId, Integer exercicio)` — executa em `@Transactional(propagation=MANDATORY)` (deve rodar dentro de transação existente)
   - `SELECT ultimo_numero FROM sequencias_documentos WHERE tipo_documento=? AND ug_id=? AND exercicio=? FOR UPDATE`
   - Se não existe: INSERT com `ultimo_numero=1`; se existe: UPDATE `ultimo_numero = ultimo_numero + 1`
   - Retorna a string formatada: `[exercício 4d][codigoUg sem traços][sequencial com LPAD 6 zeros]`
   - Tipos: `NC`, `NE`, `NL`, `OB`
3. Criar entidade JPA `BudgetAllotment` em `allotment/` com ManyToOne para `ManagingUnit`, `BudgetAction`, `InternalPlan` (nullable), `ExpenseNature`, `FundingSource`, `Ptres`
4. Criar `BudgetAllotmentRepository`:
   - `findByUgIdAndExercicio(Long ugId, Integer exercicio, Pageable p)`
   - Query nativa/JPQL `calcularTotalEmpenhado(Long dotacaoId)` — soma `valor` de NEs com status `PENDING_SETTLEMENT`, `PARTIALLY_SETTLED`, `SETTLED`, `PAID`
   - `calcularNcRecebidas(Long dotacaoId)` — soma `valor` de NCs APPROVED onde `dotacao_destino_id = dotacaoId`
   - `calcularNcCedidas(Long dotacaoId)` — soma `valor` de NCs APPROVED onde `dotacao_origem_id = dotacaoId`
5. Implementar `BudgetAllotmentService`:
   - `create(req)`, `findById(id)`, `list(filtros, pageable)`, `update(id, req)` (somente campos não-financeiros, como observações)
   - `supplement(id, valor)` — adiciona ao `valor_atualizado`; `@Transactional`
   - `getBalance(id)` — retorna DTO com: `dotacaoInicial`, `dotacaoAtualizada`, `creditosRecebidos`, `creditosCedidos`, `empenhado`, `saldoDisponivel`
6. Criar `BudgetAllotmentController` com endpoints: `GET /api/v1/allotments`, `POST`, `GET /{id}`, `PUT /{id}`, `GET /{id}/saldo`, `POST /{id}/supplement`
7. Criar entidade JPA `CreditNote` em `creditnote/` com campos conforme `04-database-design.md` e ManyToOne para as duas UGs e duas dotações
8. Implementar `CreditNoteService`:
   - `issue(req)` — valida `ugOrigem ≠ ugDestino`; gera `numeroNc` via `DocumentNumberGenerator`; status inicial `PENDING`; `@Transactional`
   - `findById(id)`, `list(filtros, pageable)`
   - `approve(id)` — verifica `status=PENDING`; calcula `saldoDisponivel` da `dotacaoOrigem`; se `valor > saldo` → lança `InsufficientBalanceException`; status → `APPROVED`; `@Transactional`
   - `cancel(id)` — apenas se `status=PENDING`; status → `CANCELLED`; se não `PENDING` → `InvalidStateTransitionException`
   - `reverse(id)` — apenas se `status=APPROVED`; verifica se há NEs ativas usando créditos da `dotacaoDestino`; se sim → lança `InvalidStateTransitionException` com detalhe; status → `REVERSED`
9. Criar exceções e adicionar handlers ao `GlobalExceptionHandler`:
   - `InsufficientBalanceException` → 422 RFC 7807
   - `InvalidStateTransitionException` → 422 RFC 7807
10. Criar `CreditNoteController` com endpoints: `GET /api/v1/credit-notes`, `POST`, `GET /{id}`, `PATCH /{id}/approve`, `PATCH /{id}/cancel`, `PATCH /{id}/reverse`
11. Escrever testes unitários `BudgetAllotmentServiceTest`:
    ```
    static final BigDecimal INITIAL_VALUE       = BigDecimal.valueOf(500_000);
    static final BigDecimal SUPPLEMENT_VALUE    = BigDecimal.valueOf(100_000);
    static final BigDecimal TOTAL_COMMITTED     = BigDecimal.valueOf(120_000);
    static final BigDecimal RECEIVED_CREDIT_NOTE = BigDecimal.valueOf(50_000);
    static final Integer    CURRENT_FISCAL_YEAR = 2025;
    ```
    - `shouldCreateAllotmentWithInitialValue()`
    - `shouldSupplementAddingValue()`
    - `shouldCalculateBalanceCorrectly()`
12. Escrever testes unitários `CreditNoteServiceTest`:
    ```
    static final BigDecimal CREDIT_NOTE_VALUE   = BigDecimal.valueOf(50_000);
    static final BigDecimal SUFFICIENT_BALANCE  = BigDecimal.valueOf(200_000);
    static final BigDecimal INSUFFICIENT_BALANCE = BigDecimal.valueOf(30_000);
    static final Long       SOURCE_UNIT_ID      = 1L;
    static final Long       TARGET_UNIT_ID      = 2L;
    ```
    - `shouldIssueCreditNoteWithPendingStatus()`
    - `shouldApproveCreditNoteWhenBalanceSufficient()`
    - `shouldThrowExceptionWhenBalanceInsufficientForCreditNote()`
    - `shouldCancelPendingCreditNote()`
    - `shouldThrowExceptionWhenCancellingApprovedCreditNote()`
    - `shouldReverseApprovedCreditNoteWithoutCommitments()`
    - `shouldThrowExceptionWhenReversingCreditNoteWithCommittedCredits()`
    - `shouldThrowExceptionWhenSourceUnitEqualsTargetUnit()`
13. Escrever testes de integração (Testcontainers) cobrindo F3-01 a F3-10 de `13-qa-process.md`
14. Criar frontend:
    - `pages/Allotment/AllotmentList.jsx`, `AllotmentForm.jsx` (com `SearchSelector` para classificações), `AllotmentBalance.jsx` (painel com os 6 campos do saldo)
    - `pages/CreditNote/CreditNoteList.jsx`, `CreditNoteForm.jsx` (com `SearchSelector` para UGs e dotações)
    - Botões de ação contextuais no detalhe da NC: "Aprovar" visível apenas em PENDING, "Cancelar" em PENDING, "Estornar" em APPROVED
    - `StatusBadge` com cores: PENDING=cinza, APPROVED=verde, CANCELLED=vermelho, REVERSED=laranja
15. Escrever testes de componente:
    ```js
    const AVAILABLE_BALANCE = 430_000;
    const CREDIT_NOTE_VALUE = 50_000;
    const ERRO_SALDO        = 'Saldo insuficiente';
    ```
    - `shouldShowBalanceWhenAllotmentSelected()`
    - `shouldShowApproveButtonOnlyWhenPending()`
    - `shouldShowConfirmationBeforeApproving()`
16. Executar `mvn verify` — cobertura `allotment` e `creditnote` ≥ 90%

#### Resultado Esperado

- Migration V4__create_budget_tables aplicada (sequências, dotações, NCs)
- Consulta de saldo retorna valores corretos
- Ciclo completo NC funcionando com validações
- Numeração automática de NC atômica (SELECT FOR UPDATE)
- Frontend com telas de dotação e NC

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `allotment` e `creditnote` ≥ 90%
2. `POST /api/v1/allotments` → 201
3. `GET /api/v1/allotments/{id}/saldo` → todos os 6 campos corretos
4. `POST /api/v1/credit-notes` → 201 com `numeroNc` no formato correto
5. `PATCH /api/v1/credit-notes/{id}/approve` com saldo suficiente → 200 `status=APPROVED`
6. `PATCH /api/v1/credit-notes/{id}/approve` com saldo insuficiente → 422 RFC 7807
7. `PATCH /api/v1/credit-notes/{id}/cancel` em NC APPROVED → 422
8. `npm test --prefix frontend` — testes de dotação e NC passam

#### Verificação Humana (Human Gate)

1. Criar dotação e verificar saldo inicial; aplicar suplementação e verificar saldo atualizado
2. Criar NC entre duas UGs; aprovar; verificar saldo na dotação origem reduzido
3. Tentar criar NC com valor acima do saldo — mensagem de erro clara
4. **QA executa os casos F3-01 a F3-10 (`13-qa-process.md`) e assina o aceite da FASE 3**

#### Critérios de Sucesso

✅ Migration V4__create_budget_tables aplicada (`sequencias_documentos`, `dotacoes_orcamentarias`, `notas_credito`)
✅ `DocumentNumberGenerator` gera números únicos atomicamente
✅ CRUD de dotações com suplementação
✅ `GET /{id}/saldo` retorna 6 campos corretos
✅ NC emitida com status PENDING e numeração automática
✅ Aprovação valida saldo
✅ Cancelamento apenas em PENDING
✅ Estorno apenas em APPROVED sem empenhos
✅ Saldo insuficiente → 422 RFC 7807
✅ Cobertura `allotment` e `creditnote` ≥ 90%
✅ Frontend funcional para dotações e NCs
✅ **QA sign-off obtido (FASE 3)**

---

### FASE-004 — Fornecedores e Notas de Empenho

#### Identidade da Fase

**ID da Fase**: FASE-004
**Nome da Fase**: Fornecedores e Notas de Empenho
**Mapeamento FR**: RF07, RF08 | **QA**: FASE 4 (`13-qa-process.md`)

#### Objetivo

É possível cadastrar fornecedores e emitir, reforçar e anular Notas de Empenho com as regras de tipo aplicadas corretamente. ORDINARY aceita apenas anulação total. ESTIMATED e GLOBAL aceitam anulação parcial. Nenhuma NE pode ser anulada se tem liquidações vigentes.

#### Dependências

FASE-003 Dotações e Notas de Crédito

#### Plano

1. Criar migração `V5__create_execution_tables.sql` com **todas** as tabelas de execução (incluindo as que serão usadas na FASE-005):
   - `fornecedores`, `notas_empenho`, `liquidacoes_empenho`, `ordens_bancarias`
   - Índices: `idx_ne_numero`, `idx_ne_dotacao`, `idx_ne_status`, `idx_ne_ug_exercicio`, `idx_fornecedor_cnpj`, `idx_nl_ne`, `idx_ob_liquidacao`

   > **Nota:** Aplicar V5 completo nesta fase — as tabelas `liquidacoes_empenho` e `ordens_bancarias` existirão no banco mas o código de serviço só será implementado na FASE-005.

2. Criar entidade JPA `Vendor` em `vendor/` com campos conforme `04-database-design.md`
3. Criar `VendorRepository`: `findByCnpj(String cnpj)`, `existsByCnpj(String cnpj)`, `findAllByStatus(String status, Pageable p)`
4. Implementar `VendorService`: `create(req)`, `findById(id)`, `list(filtros, pageable)`, `update(id, req)`, `deactivate(id)`
5. Criar `VendorController` com: `GET /api/v1/vendors`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/deactivate`
6. Criar enum `CommitmentType` com valores `ORDINARY`, `ESTIMATED`, `GLOBAL`
7. Criar enum `CommitmentStatus` com valores `PENDING_SETTLEMENT`, `PARTIALLY_SETTLED`, `SETTLED`, `PAID`, `VOIDED`
8. Criar entidade JPA `Commitment` em `commitment/` com os dois enums mapeados como `@Enumerated(EnumType.STRING)`
9. Criar `CommitmentRepository`:
   - `sumValorAtivoByDotacaoId(Long dotacaoId)` — soma valor de NEs com status ≠ `VOIDED`
   - `existsByDotacaoIdAndStatusIn(Long dotacaoId, List<CommitmentStatus> statusList)` — verifica NEs ativas
   - `existsByIdAndLiquidacoesStatus(Long neId, String statusNl)` — verifica se há NLs com status `REGISTERED`
10. Implementar `CommitmentService`:
    - `issue(req)`:
      1. Verifica `vendor.status = ACTIVE`; se não → lança `InactiveVendorException` → 422
      2. Calcula saldo disponível da dotação via `BudgetAllotmentService.getBalance()`
      3. Se `valor > saldoDisponivel` → lança `InsufficientBalanceException`
      4. Gera `numeroEmpenho` via `DocumentNumberGenerator`; status inicial `PENDING_SETTLEMENT`
      5. `@Transactional`
    - `findById(id)`, `list(filtros, pageable)`, `findByCommitment(id)` (retorna lista vazia nesta fase)
    - `reinforce(id, req)`:
      1. Verifica `status = PENDING_SETTLEMENT` ou `PARTIALLY_SETTLED`
      2. Verifica saldo disponível (novo valor total = valor atual + reforço)
      3. Atualiza `valor`; `@Transactional`
    - `voidCommitment(id, req)`:
      1. Verifica que não há NLs com `status = REGISTERED`; se sim → `InvalidStateTransitionException`
      2. Se `req.tipo = PARCIAL` e `commitmentType = ORDINARY` → lança `PartialVoidingNotAllowedException` → 422
      3. Se `req.tipo = TOTAL`: status → `VOIDED`
      4. Se `req.tipo = PARCIAL` (ESTIMATED ou GLOBAL): subtrai `req.valor` do `valor` da NE; se `valor` resultante = 0 → status `VOIDED`
      5. `@Transactional`
11. Criar `PartialVoidingNotAllowedException` → 422 e `InactiveVendorException` → 422; adicionar handlers ao `GlobalExceptionHandler`
12. Criar `CommitmentController`: `GET`, `POST`, `GET /{id}`, `POST /{id}/reinforce`, `PATCH /{id}/void`, `GET /{id}/settlements`
13. Atualizar `BudgetAllotmentRepository` para incluir as NEs no cálculo de saldo (consulta já preparada em FASE-003 mas NEs só existem agora)
14. Escrever testes unitários `VendorServiceTest`:
    ```
    static final String VALID_CNPJ      = "12345678000190";
    static final String INVALID_CNPJ    = "00000000000000";
    static final String VENDOR_NAME     = "Empresa ABC Ltda";
    ```
    - `shouldCreateVendorWithValidCNPJ()`
    - `shouldThrowExceptionWhenCNPJDuplicated()`
    - `shouldDeactivateVendor()`
    - `shouldThrowExceptionWhenVendorNotFound()`
15. Escrever testes unitários `CommitmentServiceTest`:
    ```
    static final BigDecimal COMMITMENT_VALUE        = BigDecimal.valueOf(80_000);
    static final BigDecimal SUFFICIENT_BALANCE      = BigDecimal.valueOf(500_000);
    static final BigDecimal INSUFFICIENT_BALANCE    = BigDecimal.valueOf(50_000);
    static final BigDecimal REINFORCE_VALUE         = BigDecimal.valueOf(20_000);
    static final BigDecimal PARTIAL_VOID_VALUE      = BigDecimal.valueOf(10_000);
    static final String     PROCESS_NUMBER          = "23000.001234/2025-01";
    static final String     OBJECT_DESCRIPTION      = "Aquisição de equipamentos de TI";
    static final Integer    CURRENT_FISCAL_YEAR     = 2025;
    ```
    - `shouldIssueCommitmentWhenBalanceSufficient()`
    - `shouldThrowExceptionWhenBalanceInsufficient()`
    - `shouldThrowExceptionWhenVendorInactive()`
    - `shouldSupplementCommitmentAddingValue()`
    - `shouldVoidOrdinaryCommitmentCompletely()`
    - `shouldThrowExceptionWhenPartiallyVoidingOrdinaryCommitment()`
    - `shouldPartiallyVoidEstimatedCommitment()`
    - `shouldThrowExceptionWhenVoidingCommitmentWithActiveSettlement()`
16. Escrever testes de integração (Testcontainers) cobrindo F4-01 a F4-13 de `13-qa-process.md`
17. Criar frontend:
    - `pages/Vendor/VendorList.jsx`, `VendorForm.jsx`
    - `pages/Commitment/CommitmentList.jsx`, `CommitmentForm.jsx`
    - Formulário de NE: ao selecionar a dotação via `SearchSelector`, busca e exibe saldo disponível em tempo real (`GET /api/v1/allotments/{id}/saldo`)
    - Badges de tipo (ORDINARY=azul escuro, ESTIMATED=azul, GLOBAL=roxo) e status da NE
    - Modal de confirmação para anulação com campo de valor quando tipo permite parcial
18. Executar `mvn verify` — cobertura `commitment` ≥ 90%

#### Resultado Esperado

- Migration V5__create_execution_tables aplicada (tabelas de fornecedores, NEs, NLs e OBs)
- CRUD de vendors com desativação
- NE emitida com numeração automática; reforço e anulação por tipo funcionando
- Frontend com telas de vendor e NE

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `commitment` ≥ 90%
2. `POST /api/v1/vendors` com CNPJ duplicado → 422
3. `POST /api/v1/commitments` com saldo suficiente → 201 com `numeroEmpenho`
4. `POST /api/v1/commitments` com saldo insuficiente → 422
5. `POST /api/v1/commitments/{id}/reinforce` → 200 com valor atualizado
6. `PATCH /api/v1/commitments/{id}/void` `{"tipo":"PARCIAL"}` em ORDINARY → 422
7. `PATCH /api/v1/commitments/{id}/void` `{"tipo":"TOTAL"}` em ORDINARY → 200 `status=VOIDED`
8. `npm test --prefix frontend` — testes de vendor e NE passam

#### Verificação Humana (Human Gate)

1. Cadastrar vendor; emitir NE ORDINARY; anular total; verificar saldo restaurado
2. Emitir NE ESTIMATED; anular parcialmente; verificar valor reduzido
3. Tentar emitir NE com saldo insuficiente — mensagem de erro clara
4. **QA executa os casos F4-01 a F4-13 (`13-qa-process.md`) e assina o aceite da FASE 4**

#### Critérios de Sucesso

✅ Migration V5__create_execution_tables aplicada (todas as tabelas de execução)
✅ CRUD de vendor com desativação; CNPJ duplicado → 422
✅ NE emitida com numeração automática e saldo validado
✅ Reforço de NE funcional com validação de saldo
✅ Anulação total de NE ORDINARY
✅ Anulação parcial de NE ESTIMATED/GLOBAL
✅ Anulação parcial de NE ORDINARY → 422
✅ NE com liquidação vigente não pode ser anulada → 422
✅ Cobertura `commitment` ≥ 90%
✅ Frontend funcional para vendors e NEs
✅ **QA sign-off obtido (FASE 4)**

---

### FASE-005 — Liquidação e Ordem Bancária

#### Identidade da Fase

**ID da Fase**: FASE-005
**Nome da Fase**: Liquidação e Ordem Bancária
**Mapeamento FR**: RF09, RF10 | **QA**: FASE 5 (`13-qa-process.md`)

#### Objetivo

O ciclo financeiro está completo. É possível liquidar empenhos e registrar pagamentos via OB. A propagação de status da NE (PENDING_SETTLEMENT → PARTIALLY_SETTLED → SETTLED → PAID) funciona corretamente. O fluxo ponta a ponta Dotação → NC → NE → NL → OB é operacional.

#### Dependências

FASE-004 Fornecedores e Notas de Empenho

#### Plano

> As tabelas `liquidacoes_empenho` e `ordens_bancarias` já existem no banco (criadas pela V5 na FASE-004). Esta fase apenas implementa o código de serviço.

1. Criar entidade JPA `Settlement` em `settlement/` com `@ManyToOne Commitment commitment`
2. Criar `SettlementRepository`:
   - `findByCommitmentId(Long neId, Pageable p)`
   - `sumValorByCommitmentIdAndStatus(Long neId, String status)` — total liquidado com status `REGISTERED`
   - `existsByCommitmentIdAndStatus(Long neId, String status)` — verifica NL REGISTERED (usada no bloqueio de anulação de NE)
   - `existsByIdAndPaymentOrderStatusNot(Long nlId, String statusOb)` — verifica OB vinculada não cancelada
3. Implementar `SettlementService`:
   - `register(req)`:
     1. Busca NE; verifica status ≠ `VOIDED` e ≠ `PAID`
     2. Calcula `saldoALiquidar = commitment.valor - totalJaLiquidado`
     3. Se `req.valor > saldoALiquidar` → `InsufficientBalanceException`
     4. Se `commitment.commitmentType = ORDINARY` e `req.valor < commitment.valor - totalJaLiquidado` → `PartialSettlementNotAllowedException` → 422
     5. Gera `numeroLiquidacao` via `DocumentNumberGenerator` (tipo `NL`)
     6. Após salvar NL, atualiza status da NE:
        - `totalLiquidado == commitment.valor` → `SETTLED`
        - `totalLiquidado < commitment.valor` → `PARTIALLY_SETTLED`
     7. `@Transactional`
   - `findById(id)`, `list(filtros, pageable)`
   - `reverse(id)`:
     1. Verifica `status = REGISTERED`; se não → `InvalidStateTransitionException`
     2. Verifica que não há OB com status ≠ `CANCELLED` vinculada; se sim → `InvalidStateTransitionException` "NL possui Ordem Bancária não cancelada"
     3. Status NL → `REVERSED`
     4. Recalcula status da NE com base no totalLiquidado restante
     5. `@Transactional`
4. Criar `PartialSettlementNotAllowedException` → 422; adicionar handler ao `GlobalExceptionHandler`
5. Criar `SettlementController`: `GET /api/v1/settlements`, `POST`, `GET /{id}`, `PATCH /{id}/reverse`
6. Conectar `GET /api/v1/commitments/{id}/settlements` no `CommitmentController` ao `SettlementService.findByCommitment(id)`
7. Criar entidade JPA `PaymentOrder` em `paymentorder/` com `@ManyToOne Settlement settlement`
8. Criar `PaymentOrderRepository`:
   - `existsBySettlementIdAndStatusNot(Long nlId, String status)` — verifica OB não cancelada
9. Implementar `PaymentOrderService`:
   - `issue(req)`:
     1. Busca NL; verifica `status = REGISTERED`; se não → `InvalidStateTransitionException`
     2. Verifica que não há OB não cancelada para esta NL
     3. Gera `numeroOb` via `DocumentNumberGenerator` (tipo `OB`)
     4. Status inicial `ISSUED`; `@Transactional`
   - `findById(id)`, `list(filtros, pageable)`
   - `process(id)`:
     1. Verifica `status = ISSUED`; status → `PROCESSED`
     2. Atualiza status da NE correspondente para `PAID`
     3. `@Transactional`
   - `cancel(id)`:
     1. Verifica `status = ISSUED`; status → `CANCELLED`
     2. Não altera status da NE
     3. `@Transactional`
10. Criar `PaymentOrderController`: `GET /api/v1/payment-orders`, `POST`, `GET /{id}`, `PATCH /{id}/process`, `PATCH /{id}/cancel`
11. Escrever testes unitários `SettlementServiceTest`:
    ```
    static final BigDecimal COMMITMENT_VALUE         = BigDecimal.valueOf(80_000);
    static final BigDecimal FULL_SETTLEMENT_VALUE    = BigDecimal.valueOf(80_000);
    static final BigDecimal PARTIAL_SETTLEMENT_VALUE = BigDecimal.valueOf(40_000);
    static final BigDecimal OVER_LIMIT_VALUE         = BigDecimal.valueOf(90_000);
    static final String     FISCAL_DOCUMENT          = "NF-123456";
    ```
    - `shouldRegisterFullSettlementForOrdinaryCommitment()`
    - `shouldUpdateCommitmentStatusToSettledAfterFullSettlement()`
    - `shouldRegisterPartialSettlementForEstimatedCommitment()`
    - `shouldUpdateCommitmentStatusToPartiallySettled()`
    - `shouldThrowExceptionWhenPartiallySettlingOrdinaryCommitment()`
    - `shouldThrowExceptionWhenValueExceedsPendingSettlementBalance()`
    - `shouldReverseSettlementWithoutPaymentOrder()`
    - `shouldThrowExceptionWhenReversingSettlementWithNonCancelledPaymentOrder()`
12. Escrever testes unitários `PaymentOrderServiceTest`:
    ```
    static final String     BANK               = "001";
    static final String     BRANCH             = "1234";
    static final String     DESTINATION_ACCOUNT = "56789-0";
    static final BigDecimal PAYMENT_ORDER_VALUE = BigDecimal.valueOf(80_000);
    ```
    - `shouldIssuePaymentOrderForRegisteredSettlement()`
    - `shouldThrowExceptionWhenIssuingPaymentOrderForReversedSettlement()`
    - `shouldProcessPaymentOrderUpdatingCommitmentStatusToPaid()`
    - `shouldCancelIssuedPaymentOrder()`
    - `shouldThrowExceptionWhenProcessingCancelledPaymentOrder()`
13. Escrever testes de integração (Testcontainers):
    - Cobrir F5-01 a F5-09 de `13-qa-process.md`
    - Teste `F5-10` (fluxo completo): `@Test void shouldExecuteCompleteFlowFromAllotmentToPaymentOrder()` — cria dotação, aprova NC, emite NE, registra NL, emite OB, processa OB, verifica NE com status `PAID`
14. Criar frontend:
    - `pages/Settlement/SettlementList.jsx`, `SettlementForm.jsx` (com `SearchSelector` para NEs)
    - `pages/PaymentOrder/PaymentOrderList.jsx`, `PaymentOrderForm.jsx` (com `SearchSelector` para NLs)
    - `pages/Commitment/CommitmentDetail.jsx` — exibe dados da NE + lista de NLs vinculadas + botões de ação
    - Botões contextuais por status em NL e OB
15. Executar `mvn verify` — cobertura `settlement` e `paymentorder` ≥ 90%

#### Resultado Esperado

- Código de Settlement e PaymentOrder implementado (tabelas já existiam da V5)
- Settlement valida limites e tipo da NE; propaga status corretamente
- OB emitida, processada (NE → PAID) e cancelável
- Fluxo ponta a ponta Dotação → NC → NE → NL → OB funcional
- Frontend com telas de settlement e OB

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `settlement` e `paymentorder` ≥ 90%
2. `POST /api/v1/settlements` NE ORDINARY valor total → 201; `GET /api/v1/commitments/{id}` → `status=SETTLED`
3. `POST /api/v1/settlements` NE ORDINARY valor parcial → 422
4. `POST /api/v1/settlements` valor acima do saldo → 422
5. `PATCH /api/v1/settlements/{id}/reverse` sem OB → 200 `status=REVERSED`
6. `POST /api/v1/payment-orders` para NL registrada → 201 `status=ISSUED`
7. `PATCH /api/v1/payment-orders/{id}/process` → 200; NE → `status=PAID`
8. `PATCH /api/v1/payment-orders/{id}/cancel` → 200 `status=CANCELLED`
9. Teste de integração F5-10 (fluxo completo) passa
10. `npm test --prefix frontend` — testes de NL e OB passam

#### Verificação Humana (Human Gate)

1. Executar fluxo completo: dotação → NC → NE → NL → OB → processar OB; verificar NE com status PAID
2. Tentar estornar NL com OB — mensagem de erro
3. Tentar liquidar parcialmente NE ORDINARY — mensagem de erro
4. **QA executa os casos F5-01 a F5-10 (`13-qa-process.md`) e assina o aceite da FASE 5**

#### Critérios de Sucesso

✅ Settlement total de NE ORDINARY → NE SETTLED
✅ Settlement parcial de NE ESTIMATED → NE PARTIALLY_SETTLED
✅ Settlement parcial de NE ORDINARY → 422
✅ Settlement acima do saldo → 422
✅ Reverse de NL sem OB → NE volta para PENDING_SETTLEMENT ou PARTIALLY_SETTLED
✅ Reverse de NL com OB não cancelada → 422
✅ OB emitida para NL registrada
✅ OB processada → NE PAID
✅ OB cancelada → NE não alterada
✅ Fluxo completo F5-10 funcional
✅ Cobertura `settlement` e `paymentorder` ≥ 90%
✅ Frontend funcional para NL e OB
✅ **QA sign-off obtido (FASE 5)**

---

### FASE-006 — Consultas, Dashboard e Tokens de Integração

#### Identidade da Fase

**ID da Fase**: FASE-006
**Nome da Fase**: Consultas, Dashboard e Tokens de Integração
**Mapeamento FR**: RF11–RF19 | **QA**: FASE 6 (`13-qa-process.md`)

#### Objetivo

O dashboard exibe indicadores financeiros corretos. A consulta de execução orçamentária funciona com filtros compostos. Os tokens de integração são gerados (exibidos uma vez), listados e revogados. A API é acessível com esses tokens. O rate limiting de 100 req/min por token está ativo. A auditoria AOP registra automaticamente todas as operações financeiras.

#### Dependências

FASE-005 Liquidação e Ordem Bancária

#### Plano

1. Criar migração `V6__create_audit_table.sql`:
   - Tabela `auditoria` conforme DDL em `04-database-design.md`
   - Índices `idx_auditoria_entidade`, `idx_auditoria_usuario`, `idx_auditoria_data`
2. Criar migração `V7__create_indexes.sql` com os índices que ainda não foram criados nas migrations anteriores:
   - `idx_ug_codigo`, `idx_ug_superior`, `idx_fornecedor_cnpj`, `idx_ne_numero`, `idx_ne_dotacao`, `idx_ne_status`, `idx_ne_ug_exercicio`, `idx_nc_numero`, `idx_nc_status`, `idx_nl_ne`, `idx_ob_liquidacao`

   > **Nota:** Alguns desses índices já foram criados em V2, V3, V4, V5. V7 só deve criar os que ainda não existem.

3. Criar entidade JPA `AuditEntry` em `audit/` com campos: `usuarioLogin`, `dataHora`, `operation`, `entity`, `entidadeId`, `dadosAntes` (JsonNode), `dadosDepois` (JsonNode), `ip`, `ugId`
4. Criar `AuditEntryRepository` com `findAllBy...` usando os filtros do endpoint
5. Criar a anotação `@Auditable(operation, entity)` em `audit/`
6. Implementar `AuditInterceptor` (Spring AOP `@Around`):
   - Intercepta métodos anotados com `@Auditable`
   - Captura: usuário logado via `SecurityContext`, IP via `HttpServletRequest` (injetado via `RequestContextHolder`), estado antes (chamando o repositório antes da operação), estado depois (o objeto retornado), timestamp
   - Persiste em `auditoria` via `AuditEntryRepository.save()` dentro do mesmo contexto transacional
7. Anotar com `@Auditable` os métodos de escrita nos Services:
   - `CreditNoteService`: `approve`, `cancel`, `reverse`
   - `CommitmentService`: `issue`, `reinforce`, `voidCommitment`
   - `SettlementService`: `register`, `reverse`
   - `PaymentOrderService`: `issue`, `process`, `cancel`
   - `AuthService.login` (operation: `AUTHENTICATION`, entity: `User`)
8. Criar `AuditController`: `GET /api/v1/audit-log` (filtros: `usuarioLogin`, `entity`, `operation`, `dataInicio`, `dataFim`, `ugId`; paginado)
9. Criar entidade JPA `IntegrationToken` em `token/` (tabela já existe da V1)
10. Criar `IntegrationTokenRepository`: `findByUsuarioLogin(String login)`, `findByTokenHash(String hash)`, `existsByIdAndUsuarioLogin(Long id, String login)`
11. Implementar `IntegrationTokenService`:
    - `generate(req, usuarioLogin)` — gera string aleatória `sifu_tk_` + UUID; calcula SHA-256; persiste como hash; retorna token em texto claro **apenas neste momento**
    - `list(usuarioLogin)` — retorna metadados (sem o token); ordena por `dataCriacao DESC`
    - `revoke(id, usuarioLogin)` — verifica que o token pertence ao usuário logado; status → `REVOKED`
    - `validateToken(String tokenTextoClaro)` — calcula SHA-256; busca por hash; verifica `status=ACTIVE` e `dataExpiracao > agora`; se inválido → exceção
12. Atualizar `JwtFilter` para suportar os dois tipos de token:
    - Se header `Authorization: Bearer sifu_tk_...` → autenticar via `IntegrationTokenService.validateToken()`; montar `Authentication` com o login do dono do token
    - Caso contrário → fluxo JWT existente
    - Registrar no MDC o tipo de autenticação (`SESSAO` ou `INTEGRACAO`)
13. Implementar rate limiting por token de integração:
    - Criar `RateLimitService` com `ConcurrentHashMap<String, Deque<Long>>` (por hash do token, guarda timestamps de requisições no último minuto)
    - Verificar no `JwtFilter` após autenticação de token de integração: se contagem no último minuto ≥ `RATE_LIMIT_RPM` → retornar 429 com `Retry-After: 60`
14. Criar `IntegrationTokenController`: `GET /api/v1/tokens`, `POST /api/v1/tokens`, `DELETE /api/v1/tokens/{id}`
15. Implementar `ReportService` em `report/`:
    - `budgetExecution(filtros)` — query JPQL/nativa juntando dotações, NEs e NLs:
      - `dotacaoAtualizada`: `valor_atualizado` da dotação
      - `empenhado`: soma NEs com status ≠ `VOIDED`
      - `aLiquidar`: NEs com status `PENDING_SETTLEMENT` + `PARTIALLY_SETTLED`
      - `liquidado`: soma NLs `REGISTERED`
      - `aPagar`: NEs `SETTLED` (sem OB `PROCESSED`)
      - `pago`: NEs `PAID`
      - `saldoDisponivel`: `dotacaoAtualizada - empenhado + NCs_cedidas - NCs_recebidas` (sinal invertido)
    - `dashboard(Long ugId, Integer exercicio)` — agrega os mesmos valores por UG/exercício
16. Criar `ReportController`: `GET /api/v1/reports/budget-execution`, `GET /api/v1/reports/dashboard`
17. Implementar administração de usuários em `user/`:
    - `UserService`: `list`, `create` (com BCrypt-12 na senha), `findById`, `update`, `deactivate`
    - `UserController`: `GET /api/v1/users`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/deactivate`
    - Reutilizar `UserRepository` da FASE-001
18. Escrever testes unitários `ReportServiceTest`:
    ```
    static final Long       UNIT_ID            = 1L;
    static final Integer    FISCAL_YEAR        = 2025;
    static final BigDecimal ALLOTMENT_VALUE    = BigDecimal.valueOf(1_000_000);
    static final BigDecimal COMMITTED_VALUE    = BigDecimal.valueOf(400_000);
    static final BigDecimal SETTLED_VALUE      = BigDecimal.valueOf(300_000);
    static final BigDecimal PAID_VALUE         = BigDecimal.valueOf(200_000);
    static final double     EXECUTION_PERCENTAGE = 40.0;
    ```
    - `shouldCalculateDashboardWithCorrectValues()`
    - `shouldReturnBudgetExecutionWithCorrectTotals()`
    - `shouldReturnZeroForUnitWithNoTransactions()`
19. Escrever testes unitários `IntegrationTokenServiceTest`:
    ```
    static final String TOKEN_NAME    = "Sistema de Compras";
    static final String USER_LOGIN    = "admin";
    ```
    - `shouldGenerateTokenWithHashDifferentFromPlainText()`
    - `shouldValidateActiveToken()`
    - `shouldThrowExceptionWhenValidatingRevokedToken()`
    - `shouldThrowExceptionWhenValidatingExpiredToken()`
20. Escrever testes de integração (Testcontainers) cobrindo F6-01 a F6-09 de `13-qa-process.md`
21. Escrever teste específico de rate limiting:
    ```
    static final int RPM_LIMIT         = 100;
    static final int TOTAL_REQUESTS    = 101;
    ```
    - `shouldReturn429AfterExceedingRequestLimit()`
22. Criar frontend (completar o sistema):
    - `pages/Dashboard/DashboardPage.jsx` — 4 cartões de métricas (`MetricCard`) + percentual executado
    - `pages/Report/BudgetExecutionPage.jsx` — filtros + tabela TanStack com totais
    - `pages/Token/TokensPage.jsx` — listagem de tokens + botão "Gerar Token" (modal exibe o token uma única vez com aviso explícito) + botão revogar
    - `pages/Audit/AuditLogPage.jsx` — tabela paginada com filtros por operação, entidade, usuário e período
    - `pages/User/UsersPage.jsx`, `UserForm.jsx`
    - Completar `SideMenu` com todos os 8 módulos: Dashboard, Dotações, NCs, NEs, Liquidações, OBs, Consultas, Administração (usuários, tokens, auditoria)
    - Completar `SystemHeader` com nome do usuário logado e botão "Sair"
23. Executar `mvn verify` — cobertura geral ≥ 80%; pacotes `report`, `token`, `audit` ≥ 90%

#### Resultado Esperado

- Auditoria AOP registra automaticamente todas as operações financeiras
- Tokens de integração gerados (texto claro apenas na criação), listados, revogados
- API acessível com `Authorization: Bearer sifu_tk_...`
- Rate limiting 100 req/min por token; excesso → 429
- Dashboard e consulta de execução orçamentária com valores corretos
- Frontend completo: todos os módulos acessíveis via menu lateral

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura geral ≥ 80%; críticos ≥ 90%
2. `POST /api/v1/tokens` → 201 com `token` em texto claro (apenas nesta resposta)
3. `GET /api/v1/commitments` com token de integração → 200
4. `DELETE /api/v1/tokens/{id}` → 204; usar token revogado → 401
5. `GET /api/v1/reports/dashboard?ugId=1&exercicio=2025` → DTO com 5 campos corretos
6. `GET /api/v1/reports/budget-execution?ugId=1&exercicio=2025` → totais corretos
7. 101 requisições com mesmo token em 1 minuto → última retorna 429
8. `GET /api/v1/audit-log` após operações → entradas com campos corretos
9. `npm test --prefix frontend` — testes do dashboard e tokens passam

#### Verificação Humana (Human Gate)

1. Dashboard exibe valores corretos após o fluxo completo da FASE-005
2. Gerar token — confirmar que aparece uma única vez com aviso
3. Usar token e depois revogar — confirmar 401 após revogação
4. Consultar log de auditoria — verificar que operações financeiras estão registradas
5. **QA executa os casos F6-01 a F6-09 (`13-qa-process.md`) e assina o aceite da FASE 6**

#### Critérios de Sucesso

✅ Migration V6__create_audit_table cria tabela `auditoria`
✅ AOP registra operações financeiras automaticamente
✅ `GET /api/v1/audit-log` retorna log paginado com filtros
✅ Token de integração gerado, listado e revogado
✅ API acessível com token de integração
✅ Token expirado e revogado → 401
✅ Rate limiting 100 req/min → 429 com `Retry-After`
✅ Dashboard com métricas corretas
✅ Consulta de execução orçamentária com filtros compostos
✅ Administração de usuários funcional
✅ Frontend completo (menu lateral com todos os módulos)
✅ Cobertura geral ≥ 80%; críticos ≥ 90%
✅ **QA sign-off obtido (FASE 6)**

---

### FASE-007 — Observabilidade e Deploy

#### Identidade da Fase

**ID da Fase**: FASE-007
**Nome da Fase**: Observabilidade e Deploy
**Mapeamento FR**: Nenhum (operacional) — suporta RNF-DEP, RNF-PERF

#### Objetivo

O sistema tem logging estruturado em JSON com contexto de usuário e trace, health checks via Actuator, métricas customizadas via Micrometer/Prometheus, e é totalmente containerizável via Docker Compose. O pipeline CI/CD do GitHub Actions está completo e bloqueia merges com falha de testes ou cobertura abaixo do mínimo.

#### Dependências

FASE-006 Consultas, Dashboard e Tokens de Integração

#### Plano

1. Adicionar dependência `logstash-logback-encoder` ao `pom.xml`
2. Criar `backend/src/main/resources/logback-spring.xml`:
   - Profile `default`/`dev`: appender console legível (`%d{HH:mm:ss} [%level] %logger{36} - %msg%n`)
   - Profile `prod`: appender console JSON via `LogstashEncoder` com campos: `timestamp`, `level`, `logger`, `message`, `traceId` (MDC), `usuarioLogin` (MDC)
3. Atualizar `JwtFilter` para registrar `usuarioLogin` no MDC (via `MDC.put("usuarioLogin", login)`) após autenticação bem-sucedida; limpar no `finally`
4. Gerar `traceId` por requisição no `JwtFilter` (UUID aleatório via `MDC.put("traceId", UUID.randomUUID())`); limpar no `finally`
5. Verificar e completar a configuração do Spring Actuator em `application.yml`:
   - `management.endpoints.web.exposure.include: health,info,metrics,prometheus`
   - `management.endpoint.health.show-details: when-authorized`
   - `management.endpoint.health.probes.enabled: true`
   - Verificar que `GET /actuator/health` retorna `{"status":"UP","components":{"db":{"status":"UP"}}}`
6. Adicionar dependência `micrometer-registry-prometheus` ao `pom.xml`
7. Implementar métricas customizadas via `MeterRegistry` (injetar nos Services):
   - `Counter.builder("sifu.operacoes").tag("tipo","login_sucesso").register(registry)` em `AuthService.login`
   - Contadores para: `ne_emitida`, `nc_aprovada`, `liquidacao_registrada`, `ob_processada`
   - Verificar que `GET /actuator/prometheus` exibe as métricas `sifu_operacoes_total`
8. Criar `docker-compose.test.yml` na raiz:
   - Apenas o serviço `banco` (PostgreSQL 16 com healthcheck)
   - Sem backend/frontend — usado pelo CI para os testes de integração Testcontainers
9. Verificar `backend/Dockerfile` multi-stage (criado na FASE-000):
   - Executar `docker build -t sifu-backend backend/` — imagem constrói sem erros
   - `docker run --rm sifu-backend java -jar app.jar --spring.profiles.active=test` — JAR inicia
10. Verificar `frontend/Dockerfile` (criado na FASE-000):
    - Executar `docker build -t sifu-frontend frontend/` — imagem constrói sem erros
    - Confirmar que `nginx.conf` está correto para SPA (tries `/index.html`)
11. Completar `.github/workflows/ci.yml` substituindo os stubs pelos jobs reais:

    **Job `backend-ci`:**
    1. `actions/checkout@v4`
    2. `actions/setup-java@v4` com `java-version: 21` e `distribution: temurin`
    3. Cache Maven: `~/.m2/repository`
    4. `docker compose -f docker-compose.test.yml up -d`
    5. Aguardar PostgreSQL saudável: `docker compose -f docker-compose.test.yml exec banco pg_isready -U sifu`
    6. `mvn verify -f backend/pom.xml`
    7. Falha automaticamente se JaCoCo reportar cobertura geral < 80% ou cobertura nos pacotes críticos < 90%
    8. `docker compose -f docker-compose.test.yml down`

    **Job `frontend-ci`:**
    1. `actions/checkout@v4`
    2. `actions/setup-node@v4` com `node-version: 22`
    3. Cache npm: `~/.npm`
    4. `npm ci --prefix frontend`
    5. `npm run lint --prefix frontend`
    6. `npm test --prefix frontend -- --coverage --watchAll=false`
    7. Falha se cobertura < 70%

    **Regra de branch protection:** Merge em `main` bloqueado se qualquer job falhar

12. Executar `docker compose up --build` — verificar que todos os três serviços sobem e o sistema está operacional:
    - `http://localhost:8080/actuator/health` → `{"status":"UP"}`
    - `http://localhost:8080/swagger-ui.html` → Swagger UI carrega
    - `http://localhost:5173` → Frontend carrega e login funciona
13. Executar teste de performance manual com dados de carga:
    - Seed de 500 NEs via script SQL direto
    - `GET /api/v1/notas-empenho?ugId=1&tamanho=20` → < 2s
    - `GET /api/v1/reports/budget-execution?ugId=1&exercicio=2025` → < 3s
    - Se algum target falhar, adicionar índices ou otimizar as queries JPQL
14. Revisar e atualizar `README.md` com instruções completas:
    - Pré-requisitos (Java 21, Maven, Node 22, Docker Desktop)
    - Setup: `cp .env.example .env` → editar → `docker compose up --build`
    - URLs locais (frontend, API, Swagger, Actuator)
    - Login padrão: `admin` / `Admin@123`
    - Como rodar testes: `mvn verify -f backend/pom.xml` e `npm test --prefix frontend`
    - Referência aos documentos de design em `docs/`

#### Resultado Esperado

- Logs estruturados em JSON em produção com `usuarioLogin` e `traceId` no MDC
- `GET /actuator/health` retorna saúde do banco
- `GET /actuator/prometheus` expõe métricas financeiras customizadas
- CI/CD GitHub Actions completo com thresholds de cobertura aplicados
- `docker compose up --build` sobe o sistema completo funcional
- Performance dentro dos targets: listagens < 2s, consultas < 3s
- README atualizado com instruções completas

#### Verificação Automatizada (AI Gate)

1. `docker compose up --build` — todos os serviços sobem sem erros; `sifu-banco` saudável
2. `GET /actuator/health` → `{"status":"UP","components":{"db":{"status":"UP"}}}`
3. `GET /actuator/prometheus` → contém `sifu_operacoes_total`
4. `mvn verify -f backend/pom.xml` com Testcontainers — todos os testes passam nos thresholds
5. `npm test --prefix frontend -- --coverage` — cobertura ≥ 70%
6. `docker build -t sifu-backend backend/` — imagem constrói sem erros
7. `docker build -t sifu-frontend frontend/` — imagem constrói sem erros

**Esperado:**
- Todos os serviços Docker saudáveis; métricas disponíveis; CI executa sem erros

#### Verificação Humana (Human Gate)

1. Navegar para `http://localhost:5173` — sistema completo operacional do ponto de vista do usuário
2. Executar uma operação financeira (emitir NE) e verificar que os logs JSON aparecem com `usuarioLogin` preenchido
3. Verificar pipeline CI/CD no GitHub Actions — todos os jobs passam
4. Revisar README — instruções suficientes para um desenvolvedor novo configurar o ambiente

#### Critérios de Sucesso

✅ Logging estruturado JSON com `usuarioLogin` e `traceId` no MDC
✅ `GET /actuator/health` mostra saúde do banco
✅ `GET /actuator/prometheus` expõe métricas financeiras customizadas
✅ `docker-compose.test.yml` criado para CI
✅ Dockerfiles backend e frontend constroem imagens funcionais
✅ GitHub Actions CI completo — job `backend-ci` e `frontend-ci`
✅ Thresholds de cobertura aplicados no CI (80% geral, 90% críticos, 70% frontend)
✅ Merge em `main` bloqueado se CI falhar
✅ `docker compose up --build` sobe o sistema completo
✅ Performance: listagens < 2s, consultas < 3s
✅ README atualizado com instruções completas

---

### FASE-008 — Dados de Demonstração

#### Identidade da Fase

**ID da Fase**: FASE-008
**Nome da Fase**: Dados de Demonstração
**Mapeamento FR**: Nenhum (dados)

#### Objetivo

O sistema está populado com dados fictícios porém críveis que representam um ano orçamentário completo do **Ministério da Educação (exercício 2025)**. Ao fazer login, o usuário vê um dashboard com números significativos, listas de documentos em todos os estados possíveis, e pode navegar o ciclo financeiro completo sem precisar criar dados manualmente. Os dados cobrem todas as fases do ciclo: dotações com saldo, NCs aprovadas, NEs em estados variados, liquidações e OBs processadas.

#### Dependências

FASE-007 Observabilidade e Deploy

#### Estrutura dos Dados de Demonstração

**Hierarquia de Unidades Gestoras:**
```
MIN_ED    — Ministério da Educação                    (raiz)
├── SEPS  — Secretaria de Educação Básica             (subordinada de MIN_ED)
├── SESU  — Secretaria de Educação Superior           (subordinada de MIN_ED)
└── FNDE  — Fundo Nacional de Desenvolv. da Educação (subordinada de MIN_ED)
```

**Classificações Orçamentárias:**

| Tipo | Código | Descrição |
|---|---|---|
| Ação Orçamentária | 2030 | Apoio à Educação Básica nas Redes Públicas de Ensino |
| Ação Orçamentária | 20RK | Funcionamento das Instituições Federais de Ensino Superior |
| Ação Orçamentária | 0E36 | Apoio ao Desenvolvimento da Educação Profissional |
| Plano Interno | INFRA_ESCOLA_2025 | Infraestrutura Escolar — Construção e Reforma |
| Plano Interno | BOLSAS_GRAD_2025 | Bolsas de Graduação — Prouni e FIES |
| Natureza de Despesa | 339039 | Outros Serviços de Terceiros — PJ |
| Natureza de Despesa | 339030 | Material de Consumo |
| Natureza de Despesa | 449051 | Obras e Instalações |
| Fonte de Recurso | 0100 | Recursos do Tesouro Nacional — Exercício Corrente |
| Fonte de Recurso | 0250 | Recursos Próprios — Receitas Diretamente Arrecadadas |
| PTRES | 109301 | Educação Básica — Apoio a Sistemas de Ensino |
| PTRES | 109302 | Educação Superior — Manutenção IFES |

**Dotações (exercício 2025):**

| UG | Ação | ND | Valor Inicial | Valor Atualizado |
|---|---|---|---|---|
| SEPS | 2030 | 339039 | R$ 8.000.000,00 | R$ 9.200.000,00 |
| SEPS | 2030 | 449051 | R$ 15.000.000,00 | R$ 15.000.000,00 |
| SESU | 20RK | 339039 | R$ 12.000.000,00 | R$ 13.500.000,00 |
| SESU | 20RK | 339030 | R$ 2.500.000,00 | R$ 2.500.000,00 |
| FNDE | 0E36 | 339039 | R$ 6.000.000,00 | R$ 6.800.000,00 |

**Fornecedores:**

| CNPJ | Nome | Tipo |
|---|---|---|
| 07526557000100 | Construtora Horizonte Ltda | JURIDICA |
| 33000118000179 | Papelaria Central do Brasil S.A. | JURIDICA |
| 60701190000104 | Tecbrasil Soluções em TI Ltda | JURIDICA |
| 09206050000180 | Editora Conhecimento Vivo Ltda | JURIDICA |
| 12345678000195 | Gráfica e Serviços Nacionais ME | JURIDICA |

**Notas de Crédito:**

| Número | Origem → Destino | Valor | Status |
|---|---|---|---|
| 2025MINED000001 | SEPS → FNDE | R$ 1.200.000,00 | APPROVED |
| 2025MINED000002 | SESU → SEPS | R$ 500.000,00 | APPROVED |
| 2025MINED000003 | FNDE → SESU | R$ 300.000,00 | PENDING |
| 2025MINED000004 | SEPS → SESU | R$ 800.000,00 | CANCELLED |

**Notas de Empenho (amostra representativa):**

| NE | Dotação UG | Fornecedor | Valor | Tipo | Status |
|---|---|---|---|---|---|
| 2025SEPS000001 | SEPS/2030/339039 | Tecbrasil Soluções em TI | R$ 480.000,00 | ORDINARY | PAID |
| 2025SEPS000002 | SEPS/2030/339039 | Gráfica e Serviços Nacionais | R$ 95.000,00 | ESTIMATED | SETTLED |
| 2025SEPS000003 | SEPS/2030/449051 | Construtora Horizonte | R$ 4.200.000,00 | GLOBAL | PARTIALLY_SETTLED |
| 2025SESU000001 | SESU/20RK/339039 | Papelaria Central do Brasil | R$ 750.000,00 | ORDINARY | PENDING_SETTLEMENT |
| 2025SESU000002 | SESU/20RK/339030 | Editora Conhecimento Vivo | R$ 180.000,00 | ESTIMATED | PENDING_SETTLEMENT |
| 2025FNDE000001 | FNDE/0E36/339039 | Tecbrasil Soluções em TI | R$ 320.000,00 | ORDINARY | VOIDED |
| 2025SEPS000004 | SEPS/2030/339039 | Papelaria Central do Brasil | R$ 60.000,00 | ORDINARY | PENDING_SETTLEMENT |

> Os dados são construídos de forma que o dashboard da UG `SEPS` para 2025 exiba todos os quatro indicadores preenchidos: crédito disponível, committed, settled e paid.

#### Plano

1. Criar a migração `V9__demo_data.sql` em `backend/src/main/resources/db/migration/`:
   - Proteger com condição: `INSERT ... WHERE NOT EXISTS (SELECT 1 FROM unidades_gestoras WHERE codigo_ug = 'MIN_ED')` — idempotente, não duplica dados se migration for re-executada em ambiente de desenvolvimento com banco recriado
   - Inserir UGs na ordem: `MIN_ED` primeiro (sem superior), depois as três subordinadas (com `orgao_superior_id` referenciando `MIN_ED`)
   - Inserir as 3 ações orçamentárias, 2 planos internos, 3 naturezas de despesa, 2 fontes, 2 PTRES
   - Inserir as 5 dotações com seus vínculos às classificações e UGs
   - Inserir os 5 fornecedores
   - Inserir as 4 NCs (gerando números manualmente, pois `DocumentNumberGenerator` é código de aplicação)
   - Atualizar `sequencias_documentos` para refletir os sequenciais usados
   - Inserir as 7 NEs com os estados correspondentes
   - Inserir liquidações para as NEs com status `PAID`, `SETTLED` e `PARTIALLY_SETTLED`
   - Inserir OBs para as liquidações das NEs `PAID`
   - Inserir entradas na tabela `auditoria` para as operações principais (login do admin, aprovações, emissões)

2. A migration deve seguir a ordem de inserção que respeita as FKs:
   ```
   unidades_gestoras (raiz primeiro)
   → acoes_orcamentarias
   → planos_internos (referencia acoes_orcamentarias)
   → naturezas_despesa
   → fontes_recurso
   → ptres
   → dotacoes_orcamentarias
   → sequencias_documentos (pré-popular com os sequenciais já usados)
   → fornecedores
   → notas_credito
   → notas_empenho
   → liquidacoes_empenho
   → ordens_bancarias
   → auditoria
   ```

3. Verificar que os valores são matematicamente consistentes:
   - Saldo de cada dotação = `valor_atualizado + NCs_aprovadas_recebidas − NCs_aprovadas_cedidas − NEs_ativas` deve ser ≥ 0
   - Valor total das NEs ≤ saldo da dotação correspondente
   - Valor das NLs ≤ valor das NEs vinculadas
   - Valor das OBs = valor das NLs vinculadas

4. Criar o script `backend/src/main/resources/db/demo-reset.sql`:
   - Script que apaga apenas os dados de demonstração (DELETE na ordem inversa das FKs) e re-executa V9
   - Facilita o reset em ambiente de desenvolvimento sem recriar o banco inteiro
   - Documentado no README: `docker exec sifu-banco psql -U sifu -d sifu -f /demo-reset.sql`

5. Criar arquivo `docs/implementation/dados-demonstracao.md` com:
   - Tabela de todos os usuários (login, senha, status)
   - Tabela de todas as UGs com hierarquia
   - Estado atual do sistema após a migration (saldos esperados por dotação)
   - Fluxos de navegação sugeridos para demonstração:
     - **Demo 1 — Ciclo Completo:** Abrir dotação SEPS/2030/339039 → consultar saldo → emitir nova NE → liquidar → emitir OB → processar OB → verificar dashboard
     - **Demo 2 — Nota de Crédito:** Criar NC de SESU para SEPS → aprovar → verificar impacto nos saldos
     - **Demo 3 — Dashboard:** Navegar para Dashboard com UG=SEPS/exercício=2025 → ver todos os indicadores preenchidos
     - **Demo 4 — Auditoria:** Navegar para Auditoria → ver histórico de operações pré-carregadas

6. Verificar consistência dos dados executando as queries de saldo via API após aplicar a migration:
   - `GET /api/v1/dotacoes/{id}/saldo` para cada dotação — valores devem bater com os calculados manualmente
   - `GET /api/v1/reports/dashboard?ugId={SEPS_ID}&exercicio=2025` — todos os 5 campos preenchidos com valores > 0
   - `GET /api/v1/notas-empenho?ugId={SEPS_ID}` — 4 NEs listadas em estados distintos

7. Atualizar o README com seção "Dados de Demonstração":
   - Descrever o cenário (MEC exercício 2025)
   - Listar os dados disponíveis
   - Explicar como resetar: `docker exec sifu-banco psql ...`

#### Resultado Esperado

- Migration V9 aplicada ao subir o sistema — dados de demonstração carregados automaticamente
- Dashboard de SEPS/2025 exibe: crédito disponível, total committed, settled e paid com valores reais
- Listagem de NEs mostra documentos em todos os 5 estados (PENDING_SETTLEMENT, PARTIALLY_SETTLED, SETTLED, PAID, VOIDED)
- Listagem de NCs mostra os 4 estados (PENDING, APPROVED (×2), CANCELLED — sem REVERSED para deixar esse fluxo disponível para demonstração manual)
- Log de auditoria pré-populado com operações
- Arquivo `dados-demonstracao.md` guia qualquer apresentação do sistema

#### Verificação Automatizada (AI Gate)

1. `docker compose up --build` → migration V9__demo_data aplicada sem erros
2. `GET /api/v1/allotments/{id}/saldo` para a dotação SEPS/2030/339039 → `saldoDisponivel > 0`
3. `GET /api/v1/reports/dashboard?ugId={SEPS_ID}&exercicio=2025` → todos os 5 campos > 0
4. `GET /api/v1/commitments?ugId={SEPS_ID}` → NEs em pelo menos 4 status distintos
5. `GET /api/v1/credit-notes?status=APPROVED` → pelo menos 2 NCs aprovadas
6. `GET /api/v1/audit-log` → pelo menos 10 entradas de auditoria pré-carregadas
7. Verificar idempotência: executar V9 duas vezes (simulando re-run) não duplica registros

**Esperado:**
- Sistema pronto para demonstração sem necessidade de inserção manual de dados
- Todos os valores financeiros são matematicamente consistentes

#### Verificação Humana (Human Gate)

1. Fazer login como `admin` — dashboard exibe números significativos imediatamente
2. Navegar por todos os módulos e confirmar que listas não estão vazias
3. Executar o **Demo 1** (ciclo completo) descrito em `dados-demonstracao.md` sem erros
4. Confirmar que o log de auditoria tem entradas pré-carregadas
5. Resetar os dados com o script e confirmar que o sistema retorna ao estado de demonstração

#### Critérios de Sucesso

✅ Migration V9__demo_data idempotente criada e aplicada automaticamente
✅ 4 UGs com hierarquia correta (MIN_ED raiz + 3 subordinadas)
✅ 5 dotações com valores distintos e saldos positivos
✅ 4 NCs em estados: PENDING, APPROVED (×2), CANCELLED
✅ 7 NEs em estados: PENDING_SETTLEMENT (×3), PARTIALLY_SETTLED, SETTLED, PAID, VOIDED
✅ Liquidações e OBs para NEs PAID e SETTLED
✅ Dashboard SEPS/2025 exibe todos os indicadores com valores > 0
✅ Log de auditoria pré-populado
✅ Script de reset `demo-reset.sql` funcional
✅ Arquivo `dados-demonstracao.md` documenta o cenário e os fluxos de demo

---

## Fases de Correção de Bugs

_Nenhuma fase de correção de bugs ainda. As fases de correção serão adicionadas aqui conforme necessário durante a execução._

---

## Notas

- Este plano cobre a implementação completa do SIFU — a Fase 1 do produto definida em `docs/solution-design/01-solution-overview.md`
- O plano tem **9 fases** (FASE-000 a FASE-008): 6 fases de funcionalidade com QA sign-off, 1 de infraestrutura base, 1 operacional e 1 de dados de demonstração
- Cada fase de implementação (FASE-001 a FASE-006) corresponde a uma fase de QA em `docs/solution-design/13-qa-process.md` — **nenhuma fase avança sem o sign-off do QA**
- As fases FASE-003 a FASE-005 cobrem os fluxos críticos (NC, NE, NL, OB) que exigem cobertura ≥ 90%
- A migration V5__create_execution_tables é criada **completa** na FASE-004 (incluindo tabelas de NL e OB) para evitar migrações parciais; o código de serviço de NL e OB é implementado na FASE-005
- A `sequencias_documentos` foi movida para a migration V4__create_budget_tables (em relação ao design original) pois é necessária já na FASE-003 para numeração de NCs
- Os testes de integração usam Testcontainers com PostgreSQL real — nenhum mock de banco de dados é permitido nos fluxos financeiros críticos
- **Todas as constantes de teste** devem ser declaradas no topo da classe/arquivo, antes de qualquer método, conforme a convenção em `docs/solution-design/09-testing.md`
- O frontend segue o design visual do SIAFI Web conforme `docs/solution-design/07-frontend-design.md` com a paleta DSGov (primary `#1351B4`, fonte Rawline)
- Os dados de demonstração são documentados em `docs/implementation/dados-demonstracao.md` com o cenário completo e roteiros de demo (criado na FASE-008)
- Este plano segue a estrutura de `samples/implementation/implementation-plan.md`
