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
| RN-02 | Exclusão física nunca ocorre — status `INATIVO`/`ANULADA`/`CANCELADA` |
| RN-03 | Numeração automática: `[exercício 4d][código UG sem traços][sequencial 6d]` (SELECT FOR UPDATE) |
| RN-04 | Saldo = dotação_atualizada + NCs_aprovadas_recebidas − NCs_aprovadas_cedidas − NEs_ativas |
| RN-05 | NE ORDINARIO: anulação somente total; ESTIMATIVO/GLOBAL: anulação parcial permitida |
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
2. Criar `.env.example` na raiz com os oito campos documentados em `12-deployment.md`: `DB_SENHA`, `JWT_SEGREDO`, `CORS_ORIGENS_PERMITIDAS`, `EMAIL_HOST`, `EMAIL_PORTA`, `EMAIL_USUARIO`, `EMAIL_SENHA`, `RATE_LIMIT_RPM` — todos com valores placeholder (sem credenciais reais)
3. Criar `.env` na raiz (gitignored) com valores de desenvolvimento local
4. Criar `docker-compose.yml` na raiz conforme o design em `12-deployment.md` com os três serviços: `banco` (PostgreSQL 16), `backend`, `frontend`
5. Inicializar o projeto Maven em `backend/`:
   - `groupId: br.gov.sifu`, `artifactId: sifu-backend`, Java 21, encoding UTF-8
   - Adicionar todas as dependências listadas na seção "Pacotes Backend"
   - Configurar `maven-compiler-plugin` com `-Amapstruct.defaultComponentModel=spring` e `-Amapstruct.unmappedTargetPolicy=ERROR`
   - Configurar `jacoco-maven-plugin` com threshold de cobertura geral 80% e pacotes críticos 90% (`autenticacao`, `notaempenho`, `liquidacao`, `ordembancaria`, `notacredito`)
6. Criar a estrutura completa de pacotes em `backend/src/main/java/br/gov/sifu/` conforme `05-backend-design.md`:
   - `autenticacao/` (com subpacote `dto/`)
   - `unidadegestora/` (com `dto/`)
   - `classificacao/acao/`, `classificacao/planointerno/`, `classificacao/naturezadespesa/`, `classificacao/fonterecurso/`, `classificacao/ptres/`
   - `dotacao/`, `notacredito/`, `fornecedor/`, `notaempenho/`, `liquidacao/`, `ordembancaria/`
   - `consulta/`, `usuario/`, `token/`, `auditoria/`
   - `comum/excecao/`, `comum/paginacao/`, `comum/seguranca/`, `comum/sequencia/`
7. Criar as classes base em `comum/`:
   - `ProblemDetail.java` — DTO de erro RFC 7807 com campos: `tipo`, `titulo`, `status`, `detalhe`, `instancia`
   - `TratadorGlobalExcecoes.java` — `@ControllerAdvice` com estrutura RFC 7807 (handlers serão adicionados por fase)
   - `PaginacaoUtils.java` — método `construirPageable(Integer pagina, Integer tamanho, String ordenarPor, String direcao)` com validação de `tamanho` máximo 100
   - `ConfiguracaoSeguranca.java` — `@Configuration @EnableWebSecurity` com `permitAll()` em todos os endpoints (stub provisório; substituído na FASE-001)
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
    - `componentes/comum/` — componentes reutilizáveis (`CabecalhoSistema`, `MenuLateral`, `BadgeStatus`, `ModalConfirmacao`, `TabelaDados`, `SeletorComLupa`)
    - `paginas/` — subdiretório por módulo (a popular nas fases seguintes)
    - `servicos/` — clientes HTTP via Axios (um arquivo por módulo)
    - `contextos/` — React Context (autenticação)
    - `hooks/` — custom hooks reutilizáveis
    - `utils/` — helpers de formatação (moeda BRL, datas ISO → BR)
14. Criar `frontend/src/main.jsx` com `RouterProvider` do React Router 6
15. Criar `frontend/src/roteador.jsx` com rotas stub para todos os módulos (cada rota retorna `<div>Em construção</div>` até ser implementada)
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

1. Criar migração `V1__criar_tabelas_seguranca.sql` em `backend/src/main/resources/db/migration/`:
   - Tabelas `usuarios`, `tokens_integracao`, `tokens_redefinicao_senha` conforme DDL em `04-database-design.md`
2. Criar migração `V8__dados_iniciais.sql`:
   - Inserir usuário `admin` com `login='admin'`, senha `Admin@123` hashada com BCrypt-12 e `status='ATIVO'`
   - Inserir UG raiz de exemplo: `codigo_ug='MIN_ED'`, `nome='Ministério da Educação'`
3. Criar a entidade JPA `Usuario` em `autenticacao/` com todos os campos da tabela `usuarios`; implementar `UserDetails` do Spring Security
4. Criar `UsuarioRepository` (JpaRepository) com: `findByLogin(String login)`, `findByEmail(String email)`
5. Implementar `JwtService` em `autenticacao/`:
   - `gerarToken(Usuario usuario)` — JWT com claims: `sub=login`, `nome`, `tipo=SESSAO`, `exp` = agora + 8h; assinar com HMAC-SHA256 usando `JWT_SEGREDO`
   - `extrairLogin(String token)` — extrai o subject
   - `validarToken(String token)` — verifica assinatura e expiração; lança `TokenInvalidoException` ou `TokenExpiradoException`
6. Implementar `JwtFiltro` (`OncePerRequestFilter`):
   - Extrai token do header `Authorization: Bearer <token>`
   - Autentica via `JwtService`; popula `SecurityContextHolder`
   - Ignora endpoints públicos
7. Substituir `ConfiguracaoSeguranca` stub pela configuração real:
   - Endpoints públicos: `POST /api/v1/auth/login`, `POST /api/v1/auth/recuperar-senha`, `GET /swagger-ui/**`, `GET /api-docs/**`, `GET /actuator/health`
   - Todos os demais `authenticated()`
   - Adicionar `JwtFiltro` antes de `UsernamePasswordAuthenticationFilter`
   - Configurar CORS com a origin `${CORS_ORIGENS_PERMITIDAS}`
   - Desabilitar CSRF (SPA com JWT); session stateless
8. Implementar `AutenticacaoService`:
   - `login(RequisicaoLogin req)` — verifica senha BCrypt; se inválida: incrementa `tentativasLogin`; se >= 5: seta `bloqueadoAte = agora + 15min`; se válida: zera `tentativasLogin` e retorna JWT via `JwtService`
   - `logout(String token)` — invalida sessão atual (simplificação: stateless, basta o cliente descartar o token)
   - `recuperarSenha(String email)` — busca usuário por e-mail; gera UUID, persiste em `tokens_redefinicao_senha` com `expira_em = agora + 1h`; envia e-mail via `JavaMailSender`; sempre retorna 204 (não revela existência do e-mail)
   - `redefinirSenha(RequisicaoRedefinicaoSenha req)` — busca token em `tokens_redefinicao_senha`; valida `usado=false` e `expira_em > agora`; atualiza `senha_hash` com BCrypt-12; marca token como `usado=true`
   - `alterarSenha(RequisicaoAlterarSenha req)` — verifica `senhaAtual` com BCrypt; atualiza para `novaSenha`
9. Implementar `AutenticacaoController` com os cinco endpoints de `06-api-design.md`: `POST /login`, `POST /logout`, `POST /recuperar-senha`, `PUT /redefinir-senha`, `PUT /alterar-senha`
10. Criar `ContextoSeguranca` em `comum/seguranca/` com método estático `getUsuarioLogado()` que extrai o `Usuario` do `SecurityContextHolder`
11. Criar exceções em `comum/excecao/` com handlers no `TratadorGlobalExcecoes`:
    - `CredenciaisInvalidasException` → 401 RFC 7807
    - `ContaBloqueadaException` → 423 RFC 7807 com detalhe do horário de desbloqueio
    - `TokenInvalidoException` → 401 RFC 7807
    - `TokenExpiradoException` → 401 RFC 7807
    - `EntidadeNaoEncontradaException` → 404 RFC 7807
    - Handler para `MethodArgumentNotValidException` (Bean Validation) → 400 RFC 7807 com lista de campos inválidos
12. Escrever testes unitários em `AutenticacaoServiceTest` (pacote `autenticacao`):
    ```
    // Constantes obrigatórias no topo da classe:
    static final String LOGIN_VALIDO        = "joao.silva";
    static final String SENHA_VALIDA        = "MinhaS3nha!";
    static final String SENHA_INVALIDA      = "SenhaErrada";
    static final String EMAIL_VALIDO        = "joao@orgao.gov.br";
    static final String NOME_USUARIO        = "João da Silva";
    static final int    MAX_TENTATIVAS      = 5;
    static final int    MINUTOS_BLOQUEIO    = 15;
    ```
    - `deveRetornarTokenQuandoCredenciaisValidas()`
    - `deveLancarExcecaoQuandoSenhaIncorreta()`
    - `deveIncrementarTentativasLoginQuandoSenhaIncorreta()`
    - `deveBloquerContaApos5TentativasInvalidas()`
    - `deveLancarContaBloqueadaQuandoContaEstaBloqueada()`
    - `deveGerarTokenRedefinicaoSenhaQuandoEmailValido()`
    - `deveRedefinirSenhaQuandoTokenValido()`
    - `deveLancarExcecaoQuandoTokenRedefinicaoExpirado()`
    - `deveLancarExcecaoQuandoTokenRedefinicaoJaUsado()`
13. Escrever testes de integração com Testcontainers em `AutenticacaoIntegracaoTest`:
    - Constantes: `LOGIN_ADMIN`, `SENHA_ADMIN`, `SENHA_INCORRETA`, `MAX_TENTATIVAS`, `URL_LOGIN`
    - Cobrir todos os casos do QA FASE 1 (F1-01 a F1-10 de `13-qa-process.md`)
    - Verificar que migration V8 cria o usuário admin
    - Verificar `POST /api/v1/auth/login` retorna 200 com token JWT para `admin/Admin@123`
    - Verificar `POST /api/v1/auth/login` retorna 401 para senha incorreta
    - Verificar bloqueio após 5 tentativas retorna 423
    - Verificar requisição sem token a endpoint protegido retorna 401
    - Verificar Swagger UI acessível sem token (`GET /swagger-ui.html` → 200)
14. Criar `frontend/src/servicos/autenticacaoServico.js`:
    - `login(credenciais)` — `POST /api/v1/auth/login`
    - `logout()` — `POST /api/v1/auth/logout`
    - `recuperarSenha(email)` — `POST /api/v1/auth/recuperar-senha`
    - `redefinirSenha(dados)` — `PUT /api/v1/auth/redefinir-senha`
15. Criar `frontend/src/contextos/AutenticacaoContexto.jsx`:
    - Estado: `usuario`, `token`, `autenticado`
    - Funções: `entrar(credenciais)`, `sair()`
    - Persistência do token em `localStorage`
    - Interceptor Axios global que adiciona `Authorization: Bearer <token>` a todas as requisições
    - Interceptor de resposta que chama `sair()` e redireciona para `/login` em 401
16. Criar `frontend/src/paginas/Login/PaginaLogin.jsx`:
    - Formulário com campos `login` e `senha` validados com React Hook Form + Zod
    - Exibe "Credenciais inválidas" em 401
    - Exibe "Conta bloqueada. Tente novamente em X minutos." em 423
    - Redireciona para `/` após login bem-sucedido
    - Botão desabilitado com indicador de loading durante a requisição
17. Criar `frontend/src/componentes/RotaProtegida.jsx` — wrapper que redireciona para `/login` se `!autenticado`
18. Atualizar `roteador.jsx` para envolver todas as rotas de módulos em `<RotaProtegida />`
19. Escrever testes de componente em `PaginaLogin.test.jsx`:
    ```js
    // Constantes obrigatórias no topo:
    const LOGIN_VALIDO      = 'admin';
    const SENHA_VALIDA      = 'Admin@123';
    const ERRO_CREDENCIAIS  = 'Credenciais inválidas';
    const ERRO_BLOQUEIO     = 'Conta bloqueada';
    ```
    - `deveExibirErroQuandoCredenciaisInvalidas()`
    - `deveRedirecionarParaHomeAposLoginBemSucedido()`
    - `deveDesabilitarBotaoEnquantoRequisicaoEmAndamento()`
    - `deveExibirErroDeBloqueioQuandoConta423()`
20. Executar `mvn verify -f backend/pom.xml` — cobertura do pacote `autenticacao` ≥ 90%

#### Resultado Esperado

- Migrations V1 e V8 aplicadas — tabelas de segurança e usuário admin criados
- Login retorna JWT válido; logout é funcional
- Recuperação de senha gera token (testado com mock SMTP); redefinição funciona
- Bloqueio após 5 tentativas inválidas por 15 minutos
- Todos os endpoints protegidos retornam 401 sem token
- Swagger UI acessível sem autenticação
- Frontend com página de login e rotas protegidas funcionais

#### Verificação Automatizada (AI Gate)

1. `mvn verify -f backend/pom.xml` — todos os testes passam; cobertura `autenticacao` ≥ 90%
2. `POST /api/v1/auth/login` com `{"login":"admin","senha":"Admin@123"}` → 200 com `token` JWT
3. `POST /api/v1/auth/login` com senha incorreta → 401 RFC 7807
4. Após 5 tentativas inválidas → 423 RFC 7807
5. `GET /api/v1/unidades-gestoras` sem token → 401
6. `GET /swagger-ui.html` sem token → 200
7. `npm test --prefix frontend` — testes de `PaginaLogin` passam

**Esperado:**
- Fluxo de autenticação completo funcionando; endpoints protegidos retornam 401 corretamente

#### Verificação Humana (Human Gate)

1. Navegar para `http://localhost:5173` — redireciona para `/login`
2. Login com `admin` / `Admin@123` — redireciona para tela principal
3. Swagger UI em `http://localhost:8080/swagger-ui.html` acessível sem token
4. **QA executa os casos F1-01 a F1-10 (`13-qa-process.md`) e assina o aceite da FASE 1**

#### Critérios de Sucesso

✅ Migration V1 cria `usuarios`, `tokens_integracao`, `tokens_redefinicao_senha`
✅ Migration V8 cria usuário admin com senha BCrypt-12
✅ `POST /api/v1/auth/login` retorna JWT em 200
✅ Senha errada → 401 RFC 7807
✅ Conta bloqueada após 5 tentativas → 423
✅ Logout funcional
✅ Recuperação e redefinição de senha funcionam (mock SMTP em testes)
✅ Redefinição com token expirado → 401
✅ Swagger UI acessível sem autenticação
✅ Requisição sem token → 401
✅ Cobertura `autenticacao` ≥ 90%
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

1. Criar migração `V2__criar_tabelas_estrutura.sql`:
   - Tabela `unidades_gestoras` com auto-referência `orgao_superior_id` conforme DDL em `04-database-design.md`
2. Criar migração `V3__criar_tabelas_classificacoes.sql`:
   - Tabelas `acoes_orcamentarias`, `planos_internos`, `naturezas_despesa`, `fontes_recurso`, `ptres` conforme DDL
   - Índice `idx_ug_codigo`, `idx_ug_superior`
3. Criar entidade JPA `UnidadeGestora` em `unidadegestora/` com `@ManyToOne @JoinColumn(name="orgao_superior_id") UnidadeGestora orgaoSuperior` (nullable)
4. Criar `UnidadeGestoraRepository`:
   - `findByCodigoUg(String codigo)`
   - `findByOrgaoSuperiorId(Long id)`
   - `findAllByStatus(String status, Pageable p)`
   - `existsByCodigoUg(String codigo)`
5. Criar DTOs em `unidadegestora/dto/`: `RequisicaoCriarUG`, `RequisicaoAtualizarUG`, `RespostaUG` (inclui `orgaoSuperior` simplificado: apenas `id` e `nome`)
6. Criar `UnidadeGestoraMapper` (MapStruct)
7. Implementar `UnidadeGestoraService`:
   - `criar(req)` — valida código único
   - `buscarPorId(id)` — lança `EntidadeNaoEncontradaException` se não encontrada
   - `listar(filtros, pageable)`
   - `atualizar(id, req)` — não permite alteração de `codigoUg`
   - `desativar(id)` — status → `INATIVO`
   - `listarSubordinadas(id)` — busca recursiva de subordinadas
8. Criar `UnidadeGestoraController` com endpoints: `GET /api/v1/unidades-gestoras`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/desativar`, `GET /{id}/subordinadas`
9. Para cada classificação (`AcaoOrcamentaria`, `PlanoInterno`, `NaturezaDespesa`, `FonteRecurso`, `Ptres`), criar entidade JPA, Repository, DTOs, Mapper e Service com operações: `criar`, `buscarPorId`, `listar`, `atualizar`, `desativar`, `excluir`
10. A operação `excluir` em classificações deve verificar se há dotações referenciando a classificação; se sim → lançar nova exceção `EntidadeReferenciadaException` → 422 RFC 7807 com mensagem "Classificação referenciada; não pode ser excluída"
11. Adicionar handler de `EntidadeReferenciadaException` ao `TratadorGlobalExcecoes`
12. Criar Controllers para cada classificação com padrão CRUD + `PATCH /{id}/desativar` + `DELETE /{id}` conforme `06-api-design.md`
13. Adicionar `@Auditavel` stub (anotação criada nesta fase) aos métodos de escrita dos Services — o interceptor AOP só é ativado na FASE-006; por ora a anotação existe mas não faz nada
14. Escrever testes unitários `UnidadeGestoraServiceTest`:
    ```
    static final String CODIGO_UG          = "MIN_ED";
    static final String NOME_UG            = "Ministério da Educação";
    static final String CODIGO_UG_DUP      = "MIN_ED"; // mesmo código para teste de duplicação
    static final Long   ID_UG_SUPERIOR     = 1L;
    ```
    - `deveCriarUGQuandoDadosValidos()`
    - `deveLancarExcecaoQuandoCodigoUGDuplicado()`
    - `deveDesativarUGExistente()`
    - `deveLancarExcecaoQuandoUGNaoEncontrada()`
    - `deveRetornarSubordinadasDaUG()`
    - `deveNaoPermitirAlteracaoDeCodigoUG()`
15. Escrever testes unitários análogos para pelo menos `AcaoOrcamentariaServiceTest` e `PlanoInternoServiceTest`; os demais seguem o mesmo padrão
16. Escrever testes de integração (Testcontainers) cobrindo F2-01 a F2-10 de `13-qa-process.md`
17. Criar componente reutilizável `SeletorComLupa` em `frontend/src/componentes/comum/`:
    - Input de busca com ícone de lupa
    - Abre modal/dropdown com resultado paginado
    - Emite o item selecionado via `onChange`
    - Usado em formulários de dotação, NC, NE, etc.
18. Criar `frontend/src/paginas/UnidadeGestora/` com `ListagemUG.jsx` e `FormularioUG.jsx`
19. Criar `frontend/src/paginas/Classificacao/` com `ListagemClassificacao.jsx` e `FormularioClassificacao.jsx` (componentes genéricos parametrizados por tipo)
20. Criar serviços HTTP: `unidadeGestoraServico.js`, `classificacaoServico.js`
21. Escrever testes de componente:
    ```js
    // ListagemUG.test.jsx
    const UGS_MOCK = [{ id: 1, codigoUg: 'MIN_ED', nome: 'Ministério da Educação', status: 'ATIVO' }];
    const MENSAGEM_LISTA_VAZIA = 'Nenhuma unidade gestora encontrada';
    ```
    - `deveExibirListaDeUGsCarregadas()`
    - `deveExibirMensagemSeListaVazia()`
    - `deveExibirErroDeValidacaoQuandoCampoObrigatorioVazio()`
22. Executar `mvn verify` — cobertura geral ≥ 80%

#### Resultado Esperado

- Migrations V2 e V3 aplicadas
- CRUD de UG com hierarquia; UG desativada não aparece em novos cadastros
- CRUD completo das 5 classificações com desativação e exclusão (bloqueada se referenciada)
- Frontend com listagem e formulário para UG e classificações

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura geral ≥ 80%
2. `POST /api/v1/unidades-gestoras` → 201 com `id` gerado
3. `POST /api/v1/unidades-gestoras` com `codigoUg` duplicado → 422 RFC 7807
4. `GET /api/v1/unidades-gestoras/{id}/subordinadas` → lista de subordinadas
5. `PATCH /api/v1/unidades-gestoras/{id}/desativar` → 204
6. `DELETE /api/v1/acoes-orcamentarias/{id}` em uso → 422; sem uso → 204
7. `npm test --prefix frontend` — testes de listagem e formulário passam

#### Verificação Humana (Human Gate)

1. Cadastrar UG raiz; cadastrar UG subordinada e verificar hierarquia
2. CRUD completo de uma classificação; tentar excluir classificação em uso — mensagem de erro correta
3. **QA executa os casos F2-01 a F2-10 (`13-qa-process.md`) e assina o aceite da FASE 2**

#### Critérios de Sucesso

✅ Migrations V2 e V3 aplicadas
✅ CRUD de UG com hierarquia (`orgaoSuperior`)
✅ UG desativada não aparece em novos cadastros
✅ CRUD completo de Ação Orçamentária, Plano Interno, ND, FR, PTRES
✅ Exclusão de classificação referenciada → 422
✅ Testes unitários e de integração passam
✅ `SeletorComLupa` implementado e testado
✅ Frontend: listagem e formulário para UG e classificações
✅ **QA sign-off obtido (FASE 2)**

---

### FASE-003 — Dotações e Notas de Crédito

#### Identidade da Fase

**ID da Fase**: FASE-003
**Nome da Fase**: Dotações e Notas de Crédito
**Mapeamento FR**: RF04, RF05, RF06 | **QA**: FASE 3 (`13-qa-process.md`)

#### Objetivo

É possível criar dotações orçamentárias, calcular saldos e transferir créditos entre UGs via Notas de Crédito. A máquina de estados PENDENTE → APROVADA → ESTORNADA (e o caminho CANCELADA) está completamente funcional com validações de saldo. A numeração automática de documentos funciona atomicamente.

#### Dependências

FASE-002 UGs e Classificações Orçamentárias

#### Plano

1. Criar migração `V4__criar_tabelas_orcamento.sql`:
   - Tabela `sequencias_documentos` (necessária desde esta fase para numeração de NCs)
   - Tabela `dotacoes_orcamentarias` com todos os FKs para classificações
   - Tabela `notas_credito`
   - Índice `idx_dotacao_ug_exercicio`, `idx_nc_numero`, `idx_nc_status`

   > **Nota de ajuste:** O design original (`04-database-design.md`) coloca `sequencias_documentos` em V5. Esta fase a move para V4 pois a numeração de NC já é necessária aqui. Os documentos de design não precisam ser atualizados — é uma decisão de implementação.

2. Implementar `GeradorNumeracaoDocumento` em `comum/sequencia/`:
   - `gerarNumero(String tipoDocumento, Long ugId, Integer exercicio)` — executa em `@Transactional(propagation=MANDATORY)` (deve rodar dentro de transação existente)
   - `SELECT ultimo_numero FROM sequencias_documentos WHERE tipo_documento=? AND ug_id=? AND exercicio=? FOR UPDATE`
   - Se não existe: INSERT com `ultimo_numero=1`; se existe: UPDATE `ultimo_numero = ultimo_numero + 1`
   - Retorna a string formatada: `[exercício 4d][codigoUg sem traços][sequencial com LPAD 6 zeros]`
   - Tipos: `NC`, `NE`, `NL`, `OB`
3. Criar entidade JPA `DotacaoOrcamentaria` em `dotacao/` com ManyToOne para `UnidadeGestora`, `AcaoOrcamentaria`, `PlanoInterno` (nullable), `NaturezaDespesa`, `FonteRecurso`, `Ptres`
4. Criar `DotacaoOrcamentariaRepository`:
   - `findByUgIdAndExercicio(Long ugId, Integer exercicio, Pageable p)`
   - Query nativa/JPQL `calcularTotalEmpenhado(Long dotacaoId)` — soma `valor` de NEs com status `A_LIQUIDAR`, `PARCIALMENTE_LIQUIDADA`, `LIQUIDADA`, `PAGA`
   - `calcularNcRecebidas(Long dotacaoId)` — soma `valor` de NCs APROVADAS onde `dotacao_destino_id = dotacaoId`
   - `calcularNcCedidas(Long dotacaoId)` — soma `valor` de NCs APROVADAS onde `dotacao_origem_id = dotacaoId`
5. Implementar `DotacaoOrcamentariaService`:
   - `criar(req)`, `buscarPorId(id)`, `listar(filtros, pageable)`, `atualizar(id, req)` (somente campos não-financeiros, como observações)
   - `suplementar(id, valor)` — adiciona ao `valor_atualizado`; `@Transactional`
   - `consultarSaldo(id)` — retorna DTO com: `dotacaoInicial`, `dotacaoAtualizada`, `creditosRecebidos`, `creditosCedidos`, `empenhado`, `saldoDisponivel`
6. Criar `DotacaoOrcamentariaController` com endpoints: `GET /api/v1/dotacoes`, `POST`, `GET /{id}`, `PUT /{id}`, `GET /{id}/saldo`, `POST /{id}/suplementar`
7. Criar entidade JPA `NotaCredito` em `notacredito/` com campos conforme `04-database-design.md` e ManyToOne para as duas UGs e duas dotações
8. Implementar `NotaCreditoService`:
   - `emitir(req)` — valida `ugOrigem ≠ ugDestino`; gera `numeroNc` via `GeradorNumeracaoDocumento`; status inicial `PENDENTE`; `@Transactional`
   - `buscarPorId(id)`, `listar(filtros, pageable)`
   - `aprovar(id)` — verifica `status=PENDENTE`; calcula `saldoDisponivel` da `dotacaoOrigem`; se `valor > saldo` → lança `SaldoInsuficienteException`; status → `APROVADA`; `@Transactional`
   - `cancelar(id)` — apenas se `status=PENDENTE`; status → `CANCELADA`; se não `PENDENTE` → `TransicaoEstadoInvalidaException`
   - `estornar(id)` — apenas se `status=APROVADA`; verifica se há NEs ativas usando créditos da `dotacaoDestino`; se sim → lança `TransicaoEstadoInvalidaException` com detalhe; status → `ESTORNADA`
9. Criar exceções e adicionar handlers ao `TratadorGlobalExcecoes`:
   - `SaldoInsuficienteException` → 422 RFC 7807
   - `TransicaoEstadoInvalidaException` → 422 RFC 7807
10. Criar `NotaCreditoController` com endpoints: `GET /api/v1/notas-credito`, `POST`, `GET /{id}`, `PATCH /{id}/aprovar`, `PATCH /{id}/cancelar`, `PATCH /{id}/estornar`
11. Escrever testes unitários `DotacaoOrcamentariaServiceTest`:
    ```
    static final BigDecimal VALOR_INICIAL      = BigDecimal.valueOf(500_000);
    static final BigDecimal VALOR_SUPLEMENTAR  = BigDecimal.valueOf(100_000);
    static final BigDecimal TOTAL_EMPENHADO    = BigDecimal.valueOf(120_000);
    static final BigDecimal NC_RECEBIDA        = BigDecimal.valueOf(50_000);
    static final Integer    EXERCICIO_CORRENTE = 2025;
    ```
    - `deveCriarDotacaoComValorInicial()`
    - `deveSuplementarAdicionandoValor()`
    - `deveCalcularSaldoCorretamente()`
12. Escrever testes unitários `NotaCreditoServiceTest`:
    ```
    static final BigDecimal VALOR_NC           = BigDecimal.valueOf(50_000);
    static final BigDecimal SALDO_SUFICIENTE   = BigDecimal.valueOf(200_000);
    static final BigDecimal SALDO_INSUFICIENTE = BigDecimal.valueOf(30_000);
    static final Long       UG_ORIGEM_ID       = 1L;
    static final Long       UG_DESTINO_ID      = 2L;
    ```
    - `deveEmitirNCComStatusPendente()`
    - `deveAprovarNCQuandoSaldoSuficiente()`
    - `deveLancarExcecaoQuandoSaldoInsuficienteParaNC()`
    - `deveCancelarNCPendente()`
    - `deveLancarExcecaoAoCancelarNCJaAprovada()`
    - `deveEstornarNCAprovadaSemEmpenhos()`
    - `deveLancarExcecaoAoEstornarNCComCreditosEmpenhados()`
    - `deveLancarExcecaoSeUGOrigemIgualUGDestino()`
13. Escrever testes de integração (Testcontainers) cobrindo F3-01 a F3-10 de `13-qa-process.md`
14. Criar frontend:
    - `paginas/Dotacao/ListagemDotacao.jsx`, `FormularioDotacao.jsx` (com `SeletorComLupa` para classificações), `SaldoDotacao.jsx` (painel com os 6 campos do saldo)
    - `paginas/NotaCredito/ListagemNC.jsx`, `FormularioNC.jsx` (com `SeletorComLupa` para UGs e dotações)
    - Botões de ação contextuais no detalhe da NC: "Aprovar" visível apenas em PENDENTE, "Cancelar" em PENDENTE, "Estornar" em APROVADA
    - `BadgeStatus` com cores: PENDENTE=cinza, APROVADA=verde, CANCELADA=vermelho, ESTORNADA=laranja
15. Escrever testes de componente:
    ```js
    const SALDO_DISPONIVEL = 430_000;
    const VALOR_NC         = 50_000;
    const ERRO_SALDO       = 'Saldo insuficiente';
    ```
    - `deveExibirSaldoAoSelecionarDotacao()`
    - `deveExibirBotaoAprovarApenasSePendente()`
    - `deveExibirConfirmacaoAntesDeAprovar()`
16. Executar `mvn verify` — cobertura `dotacao` e `notacredito` ≥ 90%

#### Resultado Esperado

- Migration V4 aplicada (sequências, dotações, NCs)
- Consulta de saldo retorna valores corretos
- Ciclo completo NC funcionando com validações
- Numeração automática de NC atômica (SELECT FOR UPDATE)
- Frontend com telas de dotação e NC

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `dotacao` e `notacredito` ≥ 90%
2. `POST /api/v1/dotacoes` → 201
3. `GET /api/v1/dotacoes/{id}/saldo` → todos os 6 campos corretos
4. `POST /api/v1/notas-credito` → 201 com `numeroNc` no formato correto
5. `PATCH /api/v1/notas-credito/{id}/aprovar` com saldo suficiente → 200 `status=APROVADA`
6. `PATCH /api/v1/notas-credito/{id}/aprovar` com saldo insuficiente → 422 RFC 7807
7. `PATCH /api/v1/notas-credito/{id}/cancelar` em NC APROVADA → 422
8. `npm test --prefix frontend` — testes de dotação e NC passam

#### Verificação Humana (Human Gate)

1. Criar dotação e verificar saldo inicial; aplicar suplementação e verificar saldo atualizado
2. Criar NC entre duas UGs; aprovar; verificar saldo na dotação origem reduzido
3. Tentar criar NC com valor acima do saldo — mensagem de erro clara
4. **QA executa os casos F3-01 a F3-10 (`13-qa-process.md`) e assina o aceite da FASE 3**

#### Critérios de Sucesso

✅ Migration V4 aplicada (`sequencias_documentos`, `dotacoes_orcamentarias`, `notas_credito`)
✅ `GeradorNumeracaoDocumento` gera números únicos atomicamente
✅ CRUD de dotações com suplementação
✅ `GET /{id}/saldo` retorna 6 campos corretos
✅ NC emitida com status PENDENTE e numeração automática
✅ Aprovação valida saldo
✅ Cancelamento apenas em PENDENTE
✅ Estorno apenas em APROVADA sem empenhos
✅ Saldo insuficiente → 422 RFC 7807
✅ Cobertura `dotacao` e `notacredito` ≥ 90%
✅ Frontend funcional para dotações e NCs
✅ **QA sign-off obtido (FASE 3)**

---

### FASE-004 — Fornecedores e Notas de Empenho

#### Identidade da Fase

**ID da Fase**: FASE-004
**Nome da Fase**: Fornecedores e Notas de Empenho
**Mapeamento FR**: RF07, RF08 | **QA**: FASE 4 (`13-qa-process.md`)

#### Objetivo

É possível cadastrar fornecedores e emitir, reforçar e anular Notas de Empenho com as regras de tipo aplicadas corretamente. ORDINARIO aceita apenas anulação total. ESTIMATIVO e GLOBAL aceitam anulação parcial. Nenhuma NE pode ser anulada se tem liquidações vigentes.

#### Dependências

FASE-003 Dotações e Notas de Crédito

#### Plano

1. Criar migração `V5__criar_tabelas_execucao.sql` com **todas** as tabelas de execução (incluindo as que serão usadas na FASE-005):
   - `fornecedores`, `notas_empenho`, `liquidacoes_empenho`, `ordens_bancarias`
   - Índices: `idx_ne_numero`, `idx_ne_dotacao`, `idx_ne_status`, `idx_ne_ug_exercicio`, `idx_fornecedor_cnpj`, `idx_nl_ne`, `idx_ob_liquidacao`

   > **Nota:** Aplicar V5 completo nesta fase — as tabelas `liquidacoes_empenho` e `ordens_bancarias` existirão no banco mas o código de serviço só será implementado na FASE-005.

2. Criar entidade JPA `Fornecedor` em `fornecedor/` com campos conforme `04-database-design.md`
3. Criar `FornecedorRepository`: `findByCnpj(String cnpj)`, `existsByCnpj(String cnpj)`, `findAllByStatus(String status, Pageable p)`
4. Implementar `FornecedorService`: `criar(req)`, `buscarPorId(id)`, `listar(filtros, pageable)`, `atualizar(id, req)`, `desativar(id)`
5. Criar `FornecedorController` com: `GET /api/v1/fornecedores`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/desativar`
6. Criar enum `TipoEmpenho` com valores `ORDINARIO`, `ESTIMATIVO`, `GLOBAL`
7. Criar enum `StatusNE` com valores `A_LIQUIDAR`, `PARCIALMENTE_LIQUIDADA`, `LIQUIDADA`, `PAGA`, `ANULADA`
8. Criar entidade JPA `NotaEmpenho` em `notaempenho/` com os dois enums mapeados como `@Enumerated(EnumType.STRING)`
9. Criar `NotaEmpenhoRepository`:
   - `sumValorAtivoByDotacaoId(Long dotacaoId)` — soma valor de NEs com status ≠ `ANULADA`
   - `existsByDotacaoIdAndStatusIn(Long dotacaoId, List<StatusNE> statusList)` — verifica NEs ativas
   - `existsByIdAndLiquidacoesStatus(Long neId, String statusNl)` — verifica se há NLs com status `REGISTRADA`
10. Implementar `NotaEmpenhoService`:
    - `emitir(req)`:
      1. Verifica `fornecedor.status = ATIVO`; se não → lança `FornecedorInativoException` → 422
      2. Calcula saldo disponível da dotação via `DotacaoOrcamentariaService.consultarSaldo()`
      3. Se `valor > saldoDisponivel` → lança `SaldoInsuficienteException`
      4. Gera `numeroEmpenho` via `GeradorNumeracaoDocumento`; status inicial `A_LIQUIDAR`
      5. `@Transactional`
    - `buscarPorId(id)`, `listar(filtros, pageable)`, `listarLiquidacoes(id)` (retorna lista vazia nesta fase)
    - `reforcar(id, req)`:
      1. Verifica `status = A_LIQUIDAR` ou `PARCIALMENTE_LIQUIDADA`
      2. Verifica saldo disponível (novo valor total = valor atual + reforço)
      3. Atualiza `valor`; `@Transactional`
    - `anular(id, req)`:
      1. Verifica que não há NLs com `status = REGISTRADA`; se sim → `TransicaoEstadoInvalidaException`
      2. Se `req.tipo = PARCIAL` e `tipoEmpenho = ORDINARIO` → lança `AnulacaoParcialNaoPermitidaException` → 422
      3. Se `req.tipo = TOTAL`: status → `ANULADA`
      4. Se `req.tipo = PARCIAL` (ESTIMATIVO ou GLOBAL): subtrai `req.valor` do `valor` da NE; se `valor` resultante = 0 → status `ANULADA`
      5. `@Transactional`
11. Criar `AnulacaoParcialNaoPermitidaException` → 422 e `FornecedorInativoException` → 422; adicionar handlers ao `TratadorGlobalExcecoes`
12. Criar `NotaEmpenhoController`: `GET`, `POST`, `GET /{id}`, `POST /{id}/reforco`, `PATCH /{id}/anular`, `GET /{id}/liquidacoes`
13. Atualizar `DotacaoOrcamentariaRepository` para incluir as NEs no cálculo de saldo (consulta já preparada em FASE-003 mas NEs só existem agora)
14. Escrever testes unitários `FornecedorServiceTest`:
    ```
    static final String CNPJ_VALIDO     = "12345678000190";
    static final String CNPJ_INVALIDO   = "00000000000000";
    static final String NOME_FORNECEDOR = "Empresa ABC Ltda";
    ```
    - `deveCriarFornecedorComCNPJValido()`
    - `deveLancarExcecaoQuandoCNPJDuplicado()`
    - `deveDesativarFornecedor()`
    - `deveLancarExcecaoQuandoFornecedorNaoEncontrado()`
15. Escrever testes unitários `NotaEmpenhoServiceTest`:
    ```
    static final BigDecimal VALOR_EMPENHO          = BigDecimal.valueOf(80_000);
    static final BigDecimal SALDO_SUFICIENTE        = BigDecimal.valueOf(500_000);
    static final BigDecimal SALDO_INSUFICIENTE      = BigDecimal.valueOf(50_000);
    static final BigDecimal VALOR_REFORCO           = BigDecimal.valueOf(20_000);
    static final BigDecimal VALOR_ANULACAO_PARCIAL  = BigDecimal.valueOf(10_000);
    static final String     NUMERO_PROCESSO         = "23000.001234/2025-01";
    static final String     DESCRICAO_OBJETO        = "Aquisição de equipamentos de TI";
    static final Integer    EXERCICIO_CORRENTE      = 2025;
    ```
    - `deveEmitirNEQuandoSaldoSuficiente()`
    - `deveLancarExcecaoQuandoSaldoInsuficiente()`
    - `deveLancarExcecaoQuandoFornecedorInativo()`
    - `deveReforcarNEAdicionandoValor()`
    - `deveAnularNEOrdinariaTotal()`
    - `deveLancarExcecaoAnulacaoParcialNEOrdinaria()`
    - `deveAnularNEEstimativaParcialmente()`
    - `deveLancarExcecaoAnularNEComLiquidacaoVigente()`
16. Escrever testes de integração (Testcontainers) cobrindo F4-01 a F4-13 de `13-qa-process.md`
17. Criar frontend:
    - `paginas/Fornecedor/ListagemFornecedor.jsx`, `FormularioFornecedor.jsx`
    - `paginas/NotaEmpenho/ListagemNE.jsx`, `FormularioNE.jsx`
    - Formulário de NE: ao selecionar a dotação via `SeletorComLupa`, busca e exibe saldo disponível em tempo real (`GET /api/v1/dotacoes/{id}/saldo`)
    - Badges de tipo (ORDINARIO=azul escuro, ESTIMATIVO=azul, GLOBAL=roxo) e status da NE
    - Modal de confirmação para anulação com campo de valor quando tipo permite parcial
18. Executar `mvn verify` — cobertura `notaempenho` ≥ 90%

#### Resultado Esperado

- Migration V5 aplicada (tabelas de fornecedores, NEs, NLs e OBs)
- CRUD de fornecedores com desativação
- NE emitida com numeração automática; reforço e anulação por tipo funcionando
- Frontend com telas de fornecedor e NE

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `notaempenho` ≥ 90%
2. `POST /api/v1/fornecedores` com CNPJ duplicado → 422
3. `POST /api/v1/notas-empenho` com saldo suficiente → 201 com `numeroEmpenho`
4. `POST /api/v1/notas-empenho` com saldo insuficiente → 422
5. `POST /api/v1/notas-empenho/{id}/reforco` → 200 com valor atualizado
6. `PATCH /api/v1/notas-empenho/{id}/anular` `{"tipo":"PARCIAL"}` em ORDINARIO → 422
7. `PATCH /api/v1/notas-empenho/{id}/anular` `{"tipo":"TOTAL"}` em ORDINARIO → 200 `status=ANULADA`
8. `npm test --prefix frontend` — testes de fornecedor e NE passam

#### Verificação Humana (Human Gate)

1. Cadastrar fornecedor; emitir NE ORDINARIO; anular total; verificar saldo restaurado
2. Emitir NE ESTIMATIVO; anular parcialmente; verificar valor reduzido
3. Tentar emitir NE com saldo insuficiente — mensagem de erro clara
4. **QA executa os casos F4-01 a F4-13 (`13-qa-process.md`) e assina o aceite da FASE 4**

#### Critérios de Sucesso

✅ Migration V5 aplicada (todas as tabelas de execução)
✅ CRUD de fornecedor com desativação; CNPJ duplicado → 422
✅ NE emitida com numeração automática e saldo validado
✅ Reforço de NE funcional com validação de saldo
✅ Anulação total de NE ORDINARIO
✅ Anulação parcial de NE ESTIMATIVO/GLOBAL
✅ Anulação parcial de NE ORDINARIO → 422
✅ NE com liquidação vigente não pode ser anulada → 422
✅ Cobertura `notaempenho` ≥ 90%
✅ Frontend funcional para fornecedores e NEs
✅ **QA sign-off obtido (FASE 4)**

---

### FASE-005 — Liquidação e Ordem Bancária

#### Identidade da Fase

**ID da Fase**: FASE-005
**Nome da Fase**: Liquidação e Ordem Bancária
**Mapeamento FR**: RF09, RF10 | **QA**: FASE 5 (`13-qa-process.md`)

#### Objetivo

O ciclo financeiro está completo. É possível liquidar empenhos e registrar pagamentos via OB. A propagação de status da NE (A_LIQUIDAR → PARCIALMENTE_LIQUIDADA → LIQUIDADA → PAGA) funciona corretamente. O fluxo ponta a ponta Dotação → NC → NE → NL → OB é operacional.

#### Dependências

FASE-004 Fornecedores e Notas de Empenho

#### Plano

> As tabelas `liquidacoes_empenho` e `ordens_bancarias` já existem no banco (criadas pela V5 na FASE-004). Esta fase apenas implementa o código de serviço.

1. Criar entidade JPA `LiquidacaoEmpenho` em `liquidacao/` com `@ManyToOne NotaEmpenho notaEmpenho`
2. Criar `LiquidacaoEmpenhoRepository`:
   - `findByNotaEmpenhoId(Long neId, Pageable p)`
   - `sumValorByNotaEmpenhoIdAndStatus(Long neId, String status)` — total liquidado com status `REGISTRADA`
   - `existsByNotaEmpenhoIdAndStatus(Long neId, String status)` — verifica NL REGISTRADA (usada no bloqueio de anulação de NE)
   - `existsByIdAndOrdemBancariaStatusNot(Long nlId, String statusOb)` — verifica OB vinculada não cancelada
3. Implementar `LiquidacaoEmpenhoService`:
   - `registrar(req)`:
     1. Busca NE; verifica status ≠ `ANULADA` e ≠ `PAGA`
     2. Calcula `saldoALiquidar = notaEmpenho.valor - totalJaLiquidado`
     3. Se `req.valor > saldoALiquidar` → `SaldoInsuficienteException`
     4. Se `notaEmpenho.tipoEmpenho = ORDINARIO` e `req.valor < notaEmpenho.valor - totalJaLiquidado` → `LiquidacaoParcialNaoPermitidaException` → 422
     5. Gera `numeroLiquidacao` via `GeradorNumeracaoDocumento` (tipo `NL`)
     6. Após salvar NL, atualiza status da NE:
        - `totalLiquidado == notaEmpenho.valor` → `LIQUIDADA`
        - `totalLiquidado < notaEmpenho.valor` → `PARCIALMENTE_LIQUIDADA`
     7. `@Transactional`
   - `buscarPorId(id)`, `listar(filtros, pageable)`
   - `estornar(id)`:
     1. Verifica `status = REGISTRADA`; se não → `TransicaoEstadoInvalidaException`
     2. Verifica que não há OB com status ≠ `CANCELADA` vinculada; se sim → `TransicaoEstadoInvalidaException` "NL possui Ordem Bancária não cancelada"
     3. Status NL → `ESTORNADA`
     4. Recalcula status da NE com base no totalLiquidado restante
     5. `@Transactional`
4. Criar `LiquidacaoParcialNaoPermitidaException` → 422; adicionar handler ao `TratadorGlobalExcecoes`
5. Criar `LiquidacaoEmpenhoController`: `GET /api/v1/liquidacoes`, `POST`, `GET /{id}`, `PATCH /{id}/estornar`
6. Conectar `GET /api/v1/notas-empenho/{id}/liquidacoes` no `NotaEmpenhoController` ao `LiquidacaoEmpenhoService.listarPorNE(id)`
7. Criar entidade JPA `OrdemBancaria` em `ordembancaria/` com `@ManyToOne LiquidacaoEmpenho liquidacao`
8. Criar `OrdemBancariaRepository`:
   - `existsByLiquidacaoIdAndStatusNot(Long nlId, String status)` — verifica OB não cancelada
9. Implementar `OrdemBancariaService`:
   - `emitir(req)`:
     1. Busca NL; verifica `status = REGISTRADA`; se não → `TransicaoEstadoInvalidaException`
     2. Verifica que não há OB não cancelada para esta NL
     3. Gera `numeroOb` via `GeradorNumeracaoDocumento` (tipo `OB`)
     4. Status inicial `EMITIDA`; `@Transactional`
   - `buscarPorId(id)`, `listar(filtros, pageable)`
   - `processar(id)`:
     1. Verifica `status = EMITIDA`; status → `PROCESSADA`
     2. Atualiza status da NE correspondente para `PAGA`
     3. `@Transactional`
   - `cancelar(id)`:
     1. Verifica `status = EMITIDA`; status → `CANCELADA`
     2. Não altera status da NE
     3. `@Transactional`
10. Criar `OrdemBancariaController`: `GET /api/v1/ordens-bancarias`, `POST`, `GET /{id}`, `PATCH /{id}/processar`, `PATCH /{id}/cancelar`
11. Escrever testes unitários `LiquidacaoEmpenhoServiceTest`:
    ```
    static final BigDecimal VALOR_NE                = BigDecimal.valueOf(80_000);
    static final BigDecimal VALOR_LIQUIDACAO_TOTAL  = BigDecimal.valueOf(80_000);
    static final BigDecimal VALOR_LIQUIDACAO_PARCIAL= BigDecimal.valueOf(40_000);
    static final BigDecimal VALOR_ACIMA_LIMITE      = BigDecimal.valueOf(90_000);
    static final String     DOCUMENTO_FISCAL        = "NF-123456";
    ```
    - `deveRegistrarLiquidacaoTotalDeNEOrdinaria()`
    - `deveAtualizarStatusNEParaLiquidadaAposLiquidacaoTotal()`
    - `deveRegistrarLiquidacaoParcialDeNEEstimativa()`
    - `deveAtualizarStatusNEParaParcialmenteLiquidada()`
    - `deveLancarExcecaoLiquidacaoParcialDeNEOrdinaria()`
    - `deveLancarExcecaoValorAcimaDoSaldoALiquidar()`
    - `deveEstornarNLSemOB()`
    - `deveLancarExcecaoEstornarNLComOBNaoCancelada()`
12. Escrever testes unitários `OrdemBancariaServiceTest`:
    ```
    static final String     BANCO          = "001";
    static final String     AGENCIA        = "1234";
    static final String     CONTA_DESTINO  = "56789-0";
    static final BigDecimal VALOR_OB       = BigDecimal.valueOf(80_000);
    ```
    - `deveEmitirOBParaLiquidacaoRegistrada()`
    - `deveLancarExcecaoOBParaLiquidacaoEstornada()`
    - `deveProcessarOBAtualizandoStatusNEParaPaga()`
    - `deveCancelarOBEmitida()`
    - `deveLancarExcecaoProcessarOBJaCancelada()`
13. Escrever testes de integração (Testcontainers):
    - Cobrir F5-01 a F5-09 de `13-qa-process.md`
    - Teste `F5-10` (fluxo completo): `@Test void deveExecutarFluxoCompletoDeDoatacaoAteOB()` — cria dotação, aprova NC, emite NE, registra NL, emite OB, processa OB, verifica NE com status `PAGA`
14. Criar frontend:
    - `paginas/Liquidacao/ListagemNL.jsx`, `FormularioNL.jsx` (com `SeletorComLupa` para NEs)
    - `paginas/OrdemBancaria/ListagemOB.jsx`, `FormularioOB.jsx` (com `SeletorComLupa` para NLs)
    - `paginas/NotaEmpenho/DetalheNE.jsx` — exibe dados da NE + lista de NLs vinculadas + botões de ação
    - Botões contextuais por status em NL e OB
15. Executar `mvn verify` — cobertura `liquidacao` e `ordembancaria` ≥ 90%

#### Resultado Esperado

- Código de Liquidação e OB implementado (tabelas já existiam da V5)
- Liquidação valida limites e tipo da NE; propaga status corretamente
- OB emitida, processada (NE → PAGA) e cancelável
- Fluxo ponta a ponta Dotação → NC → NE → NL → OB funcional
- Frontend com telas de liquidação e OB

#### Verificação Automatizada (AI Gate)

1. `mvn verify` — cobertura `liquidacao` e `ordembancaria` ≥ 90%
2. `POST /api/v1/liquidacoes` NE ORDINARIO valor total → 201; `GET /api/v1/notas-empenho/{id}` → `status=LIQUIDADA`
3. `POST /api/v1/liquidacoes` NE ORDINARIO valor parcial → 422
4. `POST /api/v1/liquidacoes` valor acima do saldo → 422
5. `PATCH /api/v1/liquidacoes/{id}/estornar` sem OB → 200 `status=ESTORNADA`
6. `POST /api/v1/ordens-bancarias` para NL registrada → 201 `status=EMITIDA`
7. `PATCH /api/v1/ordens-bancarias/{id}/processar` → 200; NE → `status=PAGA`
8. `PATCH /api/v1/ordens-bancarias/{id}/cancelar` → 200 `status=CANCELADA`
9. Teste de integração F5-10 (fluxo completo) passa
10. `npm test --prefix frontend` — testes de NL e OB passam

#### Verificação Humana (Human Gate)

1. Executar fluxo completo: dotação → NC → NE → NL → OB → processar OB; verificar NE com status PAGA
2. Tentar estornar NL com OB — mensagem de erro
3. Tentar liquidar parcialmente NE ORDINARIO — mensagem de erro
4. **QA executa os casos F5-01 a F5-10 (`13-qa-process.md`) e assina o aceite da FASE 5**

#### Critérios de Sucesso

✅ Liquidação total de NE ORDINARIO → NE LIQUIDADA
✅ Liquidação parcial de NE ESTIMATIVO → NE PARCIALMENTE_LIQUIDADA
✅ Liquidação parcial de NE ORDINARIO → 422
✅ Liquidação acima do saldo → 422
✅ Estorno de NL sem OB → NE volta para A_LIQUIDAR ou PARCIALMENTE_LIQUIDADA
✅ Estorno de NL com OB não cancelada → 422
✅ OB emitida para NL registrada
✅ OB processada → NE PAGA
✅ OB cancelada → NE não alterada
✅ Fluxo completo F5-10 funcional
✅ Cobertura `liquidacao` e `ordembancaria` ≥ 90%
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

1. Criar migração `V6__criar_tabela_auditoria.sql`:
   - Tabela `auditoria` conforme DDL em `04-database-design.md`
   - Índices `idx_auditoria_entidade`, `idx_auditoria_usuario`, `idx_auditoria_data`
2. Criar migração `V7__criar_indices_restantes.sql` com os índices que ainda não foram criados nas migrations anteriores:
   - `idx_ug_codigo`, `idx_ug_superior`, `idx_fornecedor_cnpj`, `idx_ne_numero`, `idx_ne_dotacao`, `idx_ne_status`, `idx_ne_ug_exercicio`, `idx_nc_numero`, `idx_nc_status`, `idx_nl_ne`, `idx_ob_liquidacao`

   > **Nota:** Alguns desses índices já foram criados em V2, V3, V4, V5. V7 só deve criar os que ainda não existem.

3. Criar entidade JPA `Auditoria` em `auditoria/` com campos: `usuarioLogin`, `dataHora`, `operacao`, `entidade`, `entidadeId`, `dadosAntes` (JsonNode), `dadosDepois` (JsonNode), `ip`, `ugId`
4. Criar `AuditoriaRepository` com `findAllBy...` usando os filtros do endpoint
5. Criar a anotação `@Auditavel(operacao, entidade)` em `auditoria/`
6. Implementar `AuditoriaInterceptor` (Spring AOP `@Around`):
   - Intercepta métodos anotados com `@Auditavel`
   - Captura: usuário logado via `ContextoSeguranca`, IP via `HttpServletRequest` (injetado via `RequestContextHolder`), estado antes (chamando o repositório antes da operação), estado depois (o objeto retornado), timestamp
   - Persiste em `auditoria` via `AuditoriaRepository.save()` dentro do mesmo contexto transacional
7. Anotar com `@Auditavel` os métodos de escrita nos Services:
   - `NotaCreditoService`: `aprovar`, `cancelar`, `estornar`
   - `NotaEmpenhoService`: `emitir`, `reforcar`, `anular`
   - `LiquidacaoEmpenhoService`: `registrar`, `estornar`
   - `OrdemBancariaService`: `emitir`, `processar`, `cancelar`
   - `AutenticacaoService.login` (operação: `AUTENTICACAO`, entidade: `Usuario`)
8. Criar `AuditoriaController`: `GET /api/v1/auditoria` (filtros: `usuarioLogin`, `entidade`, `operacao`, `dataInicio`, `dataFim`, `ugId`; paginado)
9. Criar entidade JPA `TokenIntegracao` em `token/` (tabela já existe da V1)
10. Criar `TokenIntegracaoRepository`: `findByUsuarioLogin(String login)`, `findByTokenHash(String hash)`, `existsByIdAndUsuarioLogin(Long id, String login)`
11. Implementar `TokenIntegracaoService`:
    - `gerar(req, usuarioLogin)` — gera string aleatória `sifu_tk_` + UUID; calcula SHA-256; persiste como hash; retorna token em texto claro **apenas neste momento**
    - `listar(usuarioLogin)` — retorna metadados (sem o token); ordena por `dataCriacao DESC`
    - `revogar(id, usuarioLogin)` — verifica que o token pertence ao usuário logado; status → `REVOGADO`
    - `validarToken(String tokenTextoClaro)` — calcula SHA-256; busca por hash; verifica `status=ATIVO` e `dataExpiracao > agora`; se inválido → exceção
12. Atualizar `JwtFiltro` para suportar os dois tipos de token:
    - Se header `Authorization: Bearer sifu_tk_...` → autenticar via `TokenIntegracaoService.validarToken()`; montar `Authentication` com o login do dono do token
    - Caso contrário → fluxo JWT existente
    - Registrar no MDC o tipo de autenticação (`SESSAO` ou `INTEGRACAO`)
13. Implementar rate limiting por token de integração:
    - Criar `RateLimitService` com `ConcurrentHashMap<String, Deque<Long>>` (por hash do token, guarda timestamps de requisições no último minuto)
    - Verificar no `JwtFiltro` após autenticação de token de integração: se contagem no último minuto ≥ `RATE_LIMIT_RPM` → retornar 429 com `Retry-After: 60`
14. Criar `TokenIntegracaoController`: `GET /api/v1/tokens`, `POST /api/v1/tokens`, `DELETE /api/v1/tokens/{id}`
15. Implementar `ConsultaService` em `consulta/`:
    - `execucaoOrcamentaria(filtros)` — query JPQL/nativa juntando dotações, NEs e NLs:
      - `dotacaoAtualizada`: `valor_atualizado` da dotação
      - `empenhado`: soma NEs com status ≠ `ANULADA`
      - `aLiquidar`: NEs com status `A_LIQUIDAR` + `PARCIALMENTE_LIQUIDADA`
      - `liquidado`: soma NLs `REGISTRADA`
      - `aPagar`: NEs `LIQUIDADA` (sem OB `PROCESSADA`)
      - `pago`: NEs `PAGA`
      - `saldoDisponivel`: `dotacaoAtualizada - empenhado + NCs_cedidas - NCs_recebidas` (sinal invertido)
    - `dashboard(Long ugId, Integer exercicio)` — agrega os mesmos valores por UG/exercício
16. Criar `ConsultaController`: `GET /api/v1/consultas/execucao-orcamentaria`, `GET /api/v1/consultas/dashboard`
17. Implementar administração de usuários em `usuario/`:
    - `UsuarioService`: `listar`, `criar` (com BCrypt-12 na senha), `buscarPorId`, `atualizar`, `desativar`
    - `UsuarioController`: `GET /api/v1/usuarios`, `POST`, `GET /{id}`, `PUT /{id}`, `PATCH /{id}/desativar`
    - Reutilizar `UsuarioRepository` da FASE-001
18. Escrever testes unitários `ConsultaServiceTest`:
    ```
    static final Long       UG_ID              = 1L;
    static final Integer    EXERCICIO          = 2025;
    static final BigDecimal VALOR_DOTACAO      = BigDecimal.valueOf(1_000_000);
    static final BigDecimal VALOR_EMPENHADO    = BigDecimal.valueOf(400_000);
    static final BigDecimal VALOR_LIQUIDADO    = BigDecimal.valueOf(300_000);
    static final BigDecimal VALOR_PAGO         = BigDecimal.valueOf(200_000);
    static final double     PERCENTUAL_EXEC    = 40.0;
    ```
    - `deveCalcularDashboardComValoresCorretos()`
    - `deveRetornarExecucaoOrcamentariaComTotaisCorretos()`
    - `deveRetornarZeroParaUGSemMovimentacao()`
19. Escrever testes unitários `TokenIntegracaoServiceTest`:
    ```
    static final String TOKEN_NOME    = "Sistema de Compras";
    static final String USUARIO_LOGIN = "admin";
    ```
    - `deveGerarTokenComHashDiferenteDoTexto()`
    - `deveValidarTokenAtivo()`
    - `deveLancarExcecaoAoValidarTokenRevogado()`
    - `deveLancarExcecaoAoValidarTokenExpirado()`
20. Escrever testes de integração (Testcontainers) cobrindo F6-01 a F6-09 de `13-qa-process.md`
21. Escrever teste específico de rate limiting:
    ```
    static final int LIMITE_RPM           = 100;
    static final int TOTAL_REQUISICOES    = 101;
    ```
    - `deveRetornar429AposExcederLimiteDeRequisicoes()`
22. Criar frontend (completar o sistema):
    - `paginas/Dashboard/PaginaDashboard.jsx` — 4 cartões de métricas (`CartaoMetrica`) + percentual executado
    - `paginas/Consulta/PaginaExecucaoOrcamentaria.jsx` — filtros + tabela TanStack com totais
    - `paginas/Token/PaginaTokens.jsx` — listagem de tokens + botão "Gerar Token" (modal exibe o token uma única vez com aviso explícito) + botão revogar
    - `paginas/Auditoria/PaginaAuditoria.jsx` — tabela paginada com filtros por operação, entidade, usuário e período
    - `paginas/Usuario/PaginaUsuarios.jsx`, `FormularioUsuario.jsx`
    - Completar `MenuLateral` com todos os 8 módulos: Dashboard, Dotações, NCs, NEs, Liquidações, OBs, Consultas, Administração (usuários, tokens, auditoria)
    - Completar `CabecalhoSistema` com nome do usuário logado e botão "Sair"
23. Executar `mvn verify` — cobertura geral ≥ 80%; pacotes `consulta`, `token`, `auditoria` ≥ 90%

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
3. `GET /api/v1/notas-empenho` com token de integração → 200
4. `DELETE /api/v1/tokens/{id}` → 204; usar token revogado → 401
5. `GET /api/v1/consultas/dashboard?ugId=1&exercicio=2025` → DTO com 5 campos corretos
6. `GET /api/v1/consultas/execucao-orcamentaria?ugId=1&exercicio=2025` → totais corretos
7. 101 requisições com mesmo token em 1 minuto → última retorna 429
8. `GET /api/v1/auditoria` após operações → entradas com campos corretos
9. `npm test --prefix frontend` — testes do dashboard e tokens passam

#### Verificação Humana (Human Gate)

1. Dashboard exibe valores corretos após o fluxo completo da FASE-005
2. Gerar token — confirmar que aparece uma única vez com aviso
3. Usar token e depois revogar — confirmar 401 após revogação
4. Consultar log de auditoria — verificar que operações financeiras estão registradas
5. **QA executa os casos F6-01 a F6-09 (`13-qa-process.md`) e assina o aceite da FASE 6**

#### Critérios de Sucesso

✅ Migration V6 cria tabela `auditoria`
✅ AOP registra operações financeiras automaticamente
✅ `GET /api/v1/auditoria` retorna log paginado com filtros
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
3. Atualizar `JwtFiltro` para registrar `usuarioLogin` no MDC (via `MDC.put("usuarioLogin", login)`) após autenticação bem-sucedida; limpar no `finally`
4. Gerar `traceId` por requisição no `JwtFiltro` (UUID aleatório via `MDC.put("traceId", UUID.randomUUID())`); limpar no `finally`
5. Verificar e completar a configuração do Spring Actuator em `application.yml`:
   - `management.endpoints.web.exposure.include: health,info,metrics,prometheus`
   - `management.endpoint.health.show-details: when-authorized`
   - `management.endpoint.health.probes.enabled: true`
   - Verificar que `GET /actuator/health` retorna `{"status":"UP","components":{"db":{"status":"UP"}}}`
6. Adicionar dependência `micrometer-registry-prometheus` ao `pom.xml`
7. Implementar métricas customizadas via `MeterRegistry` (injetar nos Services):
   - `Counter.builder("sifu.operacoes").tag("tipo","login_sucesso").register(registry)` em `AutenticacaoService.login`
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
    - `GET /api/v1/consultas/execucao-orcamentaria?ugId=1&exercicio=2025` → < 3s
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
| 2025MINED000001 | SEPS → FNDE | R$ 1.200.000,00 | APROVADA |
| 2025MINED000002 | SESU → SEPS | R$ 500.000,00 | APROVADA |
| 2025MINED000003 | FNDE → SESU | R$ 300.000,00 | PENDENTE |
| 2025MINED000004 | SEPS → SESU | R$ 800.000,00 | CANCELADA |

**Notas de Empenho (amostra representativa):**

| NE | Dotação UG | Fornecedor | Valor | Tipo | Status |
|---|---|---|---|---|---|
| 2025SEPS000001 | SEPS/2030/339039 | Tecbrasil Soluções em TI | R$ 480.000,00 | ORDINARIO | PAGA |
| 2025SEPS000002 | SEPS/2030/339039 | Gráfica e Serviços Nacionais | R$ 95.000,00 | ESTIMATIVO | LIQUIDADA |
| 2025SEPS000003 | SEPS/2030/449051 | Construtora Horizonte | R$ 4.200.000,00 | GLOBAL | PARCIALMENTE_LIQUIDADA |
| 2025SESU000001 | SESU/20RK/339039 | Papelaria Central do Brasil | R$ 750.000,00 | ORDINARIO | A_LIQUIDAR |
| 2025SESU000002 | SESU/20RK/339030 | Editora Conhecimento Vivo | R$ 180.000,00 | ESTIMATIVO | A_LIQUIDAR |
| 2025FNDE000001 | FNDE/0E36/339039 | Tecbrasil Soluções em TI | R$ 320.000,00 | ORDINARIO | ANULADA |
| 2025SEPS000004 | SEPS/2030/339039 | Papelaria Central do Brasil | R$ 60.000,00 | ORDINARIO | A_LIQUIDAR |

> Os dados são construídos de forma que o dashboard da UG `SEPS` para 2025 exiba todos os quatro indicadores preenchidos: crédito disponível, empenhado, liquidado e pago.

#### Plano

1. Criar a migração `V9__dados_demonstracao.sql` em `backend/src/main/resources/db/migration/`:
   - Proteger com condição: `INSERT ... WHERE NOT EXISTS (SELECT 1 FROM unidades_gestoras WHERE codigo_ug = 'MIN_ED')` — idempotente, não duplica dados se migration for re-executada em ambiente de desenvolvimento com banco recriado
   - Inserir UGs na ordem: `MIN_ED` primeiro (sem superior), depois as três subordinadas (com `orgao_superior_id` referenciando `MIN_ED`)
   - Inserir as 3 ações orçamentárias, 2 planos internos, 3 naturezas de despesa, 2 fontes, 2 PTRES
   - Inserir as 5 dotações com seus vínculos às classificações e UGs
   - Inserir os 5 fornecedores
   - Inserir as 4 NCs (gerando números manualmente, pois `GeradorNumeracaoDocumento` é código de aplicação)
   - Atualizar `sequencias_documentos` para refletir os sequenciais usados
   - Inserir as 7 NEs com os estados correspondentes
   - Inserir liquidações para as NEs com status `PAGA`, `LIQUIDADA` e `PARCIALMENTE_LIQUIDADA`
   - Inserir OBs para as liquidações das NEs `PAGA`
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
   - `GET /api/v1/consultas/dashboard?ugId={SEPS_ID}&exercicio=2025` — todos os 5 campos preenchidos com valores > 0
   - `GET /api/v1/notas-empenho?ugId={SEPS_ID}` — 4 NEs listadas em estados distintos

7. Atualizar o README com seção "Dados de Demonstração":
   - Descrever o cenário (MEC exercício 2025)
   - Listar os dados disponíveis
   - Explicar como resetar: `docker exec sifu-banco psql ...`

#### Resultado Esperado

- Migration V9 aplicada ao subir o sistema — dados de demonstração carregados automaticamente
- Dashboard de SEPS/2025 exibe: crédito disponível, total empenhado, liquidado e pago com valores reais
- Listagem de NEs mostra documentos em todos os 5 estados (A_LIQUIDAR, PARCIALMENTE_LIQUIDADA, LIQUIDADA, PAGA, ANULADA)
- Listagem de NCs mostra os 4 estados (PENDENTE, APROVADA, CANCELADA — sem ESTORNADA para deixar esse fluxo disponível para demonstração manual)
- Log de auditoria pré-populado com operações
- Arquivo `dados-demonstracao.md` guia qualquer apresentação do sistema

#### Verificação Automatizada (AI Gate)

1. `docker compose up --build` → migration V9 aplicada sem erros
2. `GET /api/v1/dotacoes/{id}/saldo` para a dotação SEPS/2030/339039 → `saldoDisponivel > 0`
3. `GET /api/v1/consultas/dashboard?ugId={SEPS_ID}&exercicio=2025` → todos os 5 campos > 0
4. `GET /api/v1/notas-empenho?ugId={SEPS_ID}` → NEs em pelo menos 4 status distintos
5. `GET /api/v1/notas-credito?status=APROVADA` → pelo menos 2 NCs aprovadas
6. `GET /api/v1/auditoria` → pelo menos 10 entradas de auditoria pré-carregadas
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

✅ Migration V9 idempotente criada e aplicada automaticamente
✅ 4 UGs com hierarquia correta (MIN_ED raiz + 3 subordinadas)
✅ 5 dotações com valores distintos e saldos positivos
✅ 4 NCs em estados: PENDENTE, APROVADA (×2), CANCELADA
✅ 7 NEs em estados: A_LIQUIDAR (×3), PARCIALMENTE_LIQUIDADA, LIQUIDADA, PAGA, ANULADA
✅ Liquidações e OBs para NEs PAGA e LIQUIDADA
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
- A migration V5 é criada **completa** na FASE-004 (incluindo tabelas de NL e OB) para evitar migrações parciais; o código de serviço de NL e OB é implementado na FASE-005
- A `sequencias_documentos` foi movida para a migration V4 (em relação ao design original) pois é necessária já na FASE-003 para numeração de NCs
- Os testes de integração usam Testcontainers com PostgreSQL real — nenhum mock de banco de dados é permitido nos fluxos financeiros críticos
- **Todas as constantes de teste** devem ser declaradas no topo da classe/arquivo, antes de qualquer método, conforme a convenção em `docs/solution-design/09-testing.md`
- O frontend segue o design visual do SIAFI Web conforme `docs/solution-design/07-frontend-design.md` com a paleta DSGov (primary `#1351B4`, fonte Rawline)
- Os dados de demonstração são documentados em `docs/implementation/dados-demonstracao.md` com o cenário completo e roteiros de demo (criado na FASE-008)
- Este plano segue a estrutura de `samples/implementation/implementation-plan.md`
