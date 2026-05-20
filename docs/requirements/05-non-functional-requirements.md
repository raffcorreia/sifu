# 05 — Requisitos Não Funcionais

## Segurança

### Autenticação e Senhas

- Login obrigatório com usuário e senha via interface web
- Proteção contra ataques de força bruta (limite de tentativas)
- Criptografia de senhas com bcrypt, PBKDF2 ou Argon2 (com salt)
- Validação de força de senha no cadastro e alteração
- Sessões com timeout apropriado e renovação de token
- Suporte obrigatório a HTTPS
- Recuperação de senha por e-mail sem exposição de dados sensíveis
- Logout completo com invalidação de sessão e token
- Tokens de acesso (JWT ou similar) para sistemas externos com expiração, renovação e revogação

### Controle de Acesso

Para fins educacionais, todos os usuários têm acesso ADMIN completo. Em produção, seria implementado um sistema de perfis:

- Operador — executa operações diárias
- Gestor Financeiro — aprova transações
- Ordenador de Despesas — autoriza despesas
- Administrador — configuração e manutenção

## Auditoria

Todas as operações financeiras devem ser auditadas com os seguintes registros:

- **usuario** — login do usuário que executou a operação
- **dataHora** — timestamp completo
- **operacao** — tipo de operação executada
- **dadosAlterados** — estado anterior e posterior da entidade
- **ip** — endereço IP da requisição
- **ugId** — unidade gestora que executou a operação

## Performance

Consultas devem responder em menos de 2 segundos para volumes de dados médios.

## Qualidade de Código

- O código deve seguir princípios de Clean Code, com nomes autoexplicativos para classes, métodos, funções e variáveis.
- Comentários em código devem ser evitados; somente são aceitáveis quando a regra de negócio, decisão arquitetural ou comportamento não forem evidentes pelo próprio código.
- Deve haver separação clara de responsabilidades, evitando mistura de regras de negócio, infraestrutura e apresentação na mesma camada.
- Funções e métodos devem ter responsabilidade única e complexidade controlada.
- Novas funcionalidades devem ser entregues com testes automatizados correspondentes (unitários e/ou integração, conforme criticidade).
- Testes automatizados devem evitar magic strings e magic numbers; valores de domínio devem ser declarados em constantes nomeadas.
- Cobertura mínima de testes no backend: **80%** geral.
- Fluxos críticos (NC, NE, Liquidação, OB, autenticação e tokens) devem ter cobertura mínima de **90%**.
- Código-fonte deve ser escrito em português (nomes de classes, métodos, funções, variáveis, arquivos técnicos e testes). Exceção: textos de interface exibidos ao usuário final também devem estar em português.
- Pull requests devem incluir evidência de execução dos testes e passar no pipeline de CI (build + testes) antes do merge.

## Confiabilidade

Operações financeiras devem ser ACID.

## Disponibilidade

99,5% de uptime.

## API e Integração

- API com versionamento explícito (ex.: `/api/v1`).
- Autenticação por token obrigatória para sistemas externos.
- Tokens de integração com expiração, revogação e rastreabilidade por usuário emissor.
- Emissão e revogação de tokens devem ser auditadas.
- Endpoints de integração devem ter limite básico de taxa (rate limiting) para reduzir abuso.
- Contrato formal e atualizado exposto via OpenAPI/Swagger.
- Documentação deve incluir endpoints, parâmetros, exemplos de requisição/resposta e códigos de erro.
- Nenhum endpoint de integração deve ir para produção sem documentação correspondente.

## Interface do Usuário

Aplicação web corporativa transacional, com as seguintes características:

- Formulários extensos
- Navegação por menus
- Forte uso de tabelas com paginação
- Filtros avançados

## Arquitetura

- **Frontend**: React ou Vue
- **Backend**: Java Spring Boot
- **API**: REST
- **Banco de dados**: PostgreSQL
- **Mensageria** (extensão futura): Kafka ou RabbitMQ
