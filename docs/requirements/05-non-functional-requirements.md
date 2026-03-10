# 05 — Requisitos Não Funcionais

## Segurança

### Autenticação e Senhas

- **Login obrigatório** com usuário e senha via interface web
- **Tela de login segura** com proteção contra ataques de força bruta (limite de tentativas)
- **Encriptação de senhas** segurada (bcrypt, PBKDF2 ou Argon2) e armazenamento salgado
- **Validação de força de senha** no cadastro / alteração de senha
- **Sessões seguras** com timeout apropriado e renovação de token
- **Suporte a HTTPS** para toda comunicação
- **Esqueci senha** e redefinição por e-mail sem expor dados
- **Logout** completo que invalida sessão e token
- **Tokens de acesso** (JWT ou similar) para sistemas externos — expiração, renovação e revogação

### Controle de Acesso

**Nota:** Para fins educacionais, todos os usuários têm acesso ADMIN completo. Em produção, seria implementado um sistema de perfis:

- Operador - Executa operações diárias
- Gestor Financeiro - Aprova transações
- Ordenador de Despesas - Autoriza despesas
- Administrador - Configuração e manutenção

## Auditoria

Todas as operações financeiras devem ser auditadas com os seguintes registros:

- **usuário** (login do usuário que executou a operação)
- **data e hora** (timestamp completo)
- **operacao** (tipo de operação executada)
- **dados alterados** (antes e depois da mudança)
- **ip** (endereço IP da requisição)
- **ug_id** (unidade gestora que executou a operação)

A auditoria é essencial para rastrear todas as operações financeiras e garantir conformidade.

## Performance

Consultas devem responder em < 2 segundos para datasets médios.

## Qualidade de Código

- O código deve seguir princípios de Clean Code, com nomes autoexplicativos para classes, métodos, funções e variáveis.
- Comentários em código devem ser evitados; somente são aceitáveis em casos raros, quando a regra de negócio, decisão arquitetural ou comportamento não forem evidentes pelo próprio código.
- Deve haver separação clara de responsabilidades (Separation of Concerns), evitando mistura de regras de negócio, infraestrutura e apresentação na mesma camada.
- Funções e métodos devem manter complexidade controlada e responsabilidade única sempre que possível.
- Novas funcionalidades devem ser entregues com testes automatizados correspondentes (unitários e/ou integração, conforme criticidade).
- Testes automatizados devem evitar literals soltos (`magic strings` e `magic numbers`) no corpo dos testes; valores de domínio devem ser declarados em constantes nomeadas no início do teste/arquivo.
- Cobertura mínima de testes automatizados deve ser de 80% no backend.
- Fluxos financeiros críticos (NC, NE, Liquidação, OB, autenticação e emissão/revogação de token) devem possuir cobertura de testes de 90% ou superior.
- Código-fonte deve ser escrito em inglês (nomes de classes, métodos, funções, variáveis, arquivos técnicos e testes). Exceção: textos de interface exibidos ao usuário final devem estar em português.
- Pull requests devem incluir evidência de execução de testes e passar no pipeline de CI (build + testes) antes de merge.

## Confiabilidade

Operações financeiras devem ser ACID.

## Disponibilidade

99.5% uptime.

## API e Integração

- API deve seguir versionamento explícito (ex.: `/api/v1`).
- API deve exigir autenticação por token para sistemas externos.
- Tokens de integração devem possuir expiração, possibilidade de revogação e rastreabilidade por usuário emissor.
- A emissão e a revogação de tokens devem ser auditadas.
- Endpoints de integração devem possuir limite básico de taxa para reduzir abuso.
- API deve expor contrato formal e atualizado (OpenAPI/Swagger ou equivalente, a ser definido na fase de design).
- Documentação de API deve incluir endpoints, parâmetros, exemplos de requisição/resposta e códigos de erro.
- Não deve haver endpoint de integração em produção sem documentação correspondente.

## 15. Interface do Usuário

### Tipo de Interface

Aplicação **Web corporativa transacional**.

**Características:**

- formulários extensos
- navegação por menus
- forte uso de tabelas
- filtros avançados

## 16. Tecnologias Prováveis do SIAFI Web

Não há documentação pública completa, mas pela análise de sistemas federais similares é provável que utilize:

### Backend

Possíveis tecnologias:

- Java
- Spring
- EJB
- Mainframe integration
- SOAP services

### Frontend

Arquitetura típica de sistemas governamentais:

- HTML
- CSS
- JavaScript
- JSP ou JSF

**Bibliotecas possíveis:**

- PrimeFaces
- RichFaces
- jQuery
- Bootstrap

**Interface padrão:**

- tabelas com paginação
- filtros avançados
- formulários estruturados
- menus laterais

## 17. Arquitetura Sugerida para Novo Sistema

Arquitetura moderna recomendada:

- **Frontend**: React / Vue
- **Backend**: Java Spring Boot
- **API**: REST
- **Banco**: PostgreSQL
- **Mensageria**: Kafka / RabbitMQ