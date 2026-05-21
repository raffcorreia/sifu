# 09 — Estratégia de Testes

> **Camada:** Experience | **Depende de:** 05-backend-design, 07-frontend-design

## Metas de Cobertura

| Escopo | Cobertura Mínima |
|---|---|
| Backend geral | 80% |
| Fluxos críticos (NC, NE, NL, OB, auth, tokens) | 90% |
| Frontend (componentes principais) | 70% |

## Backend

### Testes Unitários (JUnit 5 + Mockito)

Testam a camada de Service em isolamento. Repositórios são mockados com Mockito.

**O que testar por Service:**
- Caminho feliz (operação bem-sucedida)
- Validações de negócio (saldo insuficiente, estado inválido, etc.)
- Transições de estado corretas
- Geração de número de documento

**Convenções de nomenclatura:**
```java
// padrão: dado_quando_entao
@Test
void deveEmitirEmpenhoQuandoSaldoSuficiente() { ... }

@Test
void deveLancarExcecaoQuandoSaldoInsuficiente() { ... }

@Test
void deveAnularEmpenhoOrdinariototalmenteApenasComAnulacaoTotal() { ... }
```

**Constantes nomeadas em vez de magic values:**
```java
// Correto
static final BigDecimal VALOR_EMPENHO = BigDecimal.valueOf(80_000);
static final BigDecimal SALDO_INSUFICIENTE = BigDecimal.valueOf(50_000);

// Errado
var ne = new NotaEmpenho(80000, ...); // magic number
```

### Testes de Integração (Testcontainers + PostgreSQL)

Testam Controllers + Services + Repositórios com banco real em container Docker.

- Usam `@SpringBootTest` + `@AutoConfigureMockMvc`
- Cada teste limpa o banco via `@Transactional` com rollback ou scripts SQL
- Testcontainers inicia um PostgreSQL real — sem H2 ou mocks de banco

**Casos obrigatórios por fluxo crítico:**

| Fluxo | Casos de Teste |
|---|---|
| Emissão NE | saldo suficiente, saldo insuficiente, dotação inativa, fornecedor inativo |
| Anulação NE | total sem liquidações, total com liquidação existente (deve falhar), parcial em ordinário (deve falhar) |
| Reforço NE | dentro do exercício, exercício diferente (deve falhar) |
| Aprovação NC | saldo disponível, saldo insuficiente, NC já aprovada (deve falhar) |
| Estorno NC | créditos não empenhados, créditos já empenhados no destino (deve falhar) |
| Registro NL | valor dentro do limite, valor acima do empenhado (deve falhar), ordinário com liquidação parcial (deve falhar) |
| Estorno NL | sem OB, com OB existente (deve falhar) |
| Emissão OB | liquidação registrada, liquidação já paga (deve falhar) |
| Login | credenciais válidas, inválidas, conta bloqueada |
| Token | geração, revogação, uso após revogação (deve falhar) |

### Testes de Validação de API

Usando `MockMvc`, verificam:
- Campos obrigatórios retornam 400 com mensagem de erro
- Autenticação ausente retorna 401
- Token expirado retorna 401
- Respostas de erro seguem o formato RFC 7807

## Frontend

### Testes de Componente (Jest + React Testing Library)

Testam componentes React em isolamento, com mock do cliente HTTP (Axios).

**O que testar:**
- Renderização correta de dados recebidos via props
- Submissão de formulário com dados válidos chama a API corretamente
- Mensagem de erro exibida quando a API retorna erro
- Campos obrigatórios bloqueiam o submit quando vazios
- Confirmação modal para ações destrutivas

**Exemplo:**
```js
// Convenção: descreve o comportamento do usuário
test('deve exibir erro quando saldo for insuficiente', async () => { ... });
test('deve desabilitar botão durante o envio', async () => { ... });
test('deve exibir modal de confirmação ao cancelar NC', async () => { ... });
```

### Testes End-to-End (Fase 2)

Playwright ou Cypress para fluxos completos — fora do escopo da Fase 1.

## CI Pipeline

```yaml
# .github/workflows/ci.yml — estrutura

jobs:
  backend:
    - checkout
    - setup-java 21
    - mvn verify          ← compila + testes + cobertura (JaCoCo)
    - upload coverage report
    - falha se cobertura < 80% (geral) ou < 90% (pacotes críticos)

  frontend:
    - checkout
    - setup-node 22
    - npm ci
    - npm run lint
    - npm test -- --coverage
    - falha se cobertura < 70%
```

Pull requests só podem ser mergeados após CI verde.
