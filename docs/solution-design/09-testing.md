# 09 — Estratégia de Testes

> **Camada:** Experience | **Depende de:** 05-backend-design, 07-frontend-design

## Metas de Cobertura

| Escopo | Cobertura Mínima |
|---|---|
| Backend geral | 80% |
| Fluxos críticos (NC, NE, NL, OB, auth, tokens) | 90% |
| Frontend (componentes principais) | 70% |

## Definição de Pronto (Definition of Done)

Uma funcionalidade só está concluída quando:

1. Código implementado e revisado
2. **Testes automatizados escritos ou atualizados** cobrindo os fluxos da funcionalidade
3. CI passando (build + testes + cobertura)
4. Documentação correspondente criada/atualizada

Testes **nunca** são escritos após a conclusão da fase — fazem parte da entrega de cada funcionalidade.

---

## Convenções de Testes

### Constantes no Topo

**Todos os valores usados nos testes devem ser declarados como constantes nomeadas no topo da classe ou arquivo**, antes de qualquer método. Nenhum valor literal (magic string, magic number) pode aparecer no corpo dos testes.

**Java — correto:**
```java
class NotaEmpenhoServiceTest {

    static final BigDecimal VALOR_EMPENHO          = BigDecimal.valueOf(80_000);
    static final BigDecimal SALDO_SUFICIENTE        = BigDecimal.valueOf(500_000);
    static final BigDecimal SALDO_INSUFICIENTE      = BigDecimal.valueOf(50_000);
    static final BigDecimal VALOR_REFORCO           = BigDecimal.valueOf(20_000);
    static final String     NUMERO_PROCESSO         = "23000.001234/2025-01";
    static final String     DESCRICAO_OBJETO        = "Aquisição de equipamentos de TI";
    static final Integer    EXERCICIO_CORRENTE      = 2025;
    static final Integer    EXERCICIO_ANTERIOR      = 2024;

    @Test
    void deveEmitirEmpenhoQuandoSaldoSuficiente() {
        var dotacao = dotacaoComSaldo(SALDO_SUFICIENTE);
        var requisicao = requisicaoNE(VALOR_EMPENHO, NUMERO_PROCESSO);
        // ...
    }
}
```

**Java — errado:**
```java
@Test
void deveEmitirEmpenho() {
    var dotacao = dotacaoComSaldo(new BigDecimal("500000")); // ← magic number
    var requisicao = new RequisicaoNE(80000.0, "23000.001234/2025-01"); // ← magic values
}
```

**JavaScript/TypeScript — correto:**
```js
const VALOR_EMPENHO        = 80_000;
const SALDO_SUFICIENTE     = 500_000;
const NOME_FORNECEDOR      = 'Empresa ABC Ltda';
const CNPJ_FORNECEDOR      = '12345678000190';
const MENSAGEM_SALDO_INSUF = 'Saldo insuficiente';

test('deve exibir erro quando saldo for insuficiente', () => {
    // usa constantes acima
});
```

### Nomenclatura de Testes

Padrão: `deve[Resultado]Quando[Condição]`

```java
// Backend
void deveEmitirEmpenhoQuandoSaldoSuficiente()
void deveLancarExcecaoQuandoSaldoInsuficiente()
void deveImpedirAnulacaoParcialDeEmpenhoOrdinario()
void deveImpedirEstornoNLComOBVinculada()
void deveBloquerContaApos5TentativasInvalidas()

// Frontend
test('deve exibir saldo disponível ao selecionar dotação')
test('deve desabilitar botão Emitir durante requisição')
test('deve exibir modal de confirmação ao anular empenho')
test('deve limpar formulário após emissão bem-sucedida')
```

---

## Backend

### Testes Unitários (JUnit 5 + Mockito)

Testam a camada de Service em isolamento. Repositórios são mockados com Mockito.

**Estrutura de cada teste unitário:**
```java
class NotaEmpenhoServiceTest {

    // 1. Constantes no topo
    static final BigDecimal VALOR_EMPENHO     = BigDecimal.valueOf(80_000);
    static final BigDecimal SALDO_SUFICIENTE  = BigDecimal.valueOf(500_000);

    // 2. Mocks e instância do service
    @Mock NotaEmpenhoRepository repositorio;
    @Mock DotacaoService dotacaoService;
    @InjectMocks NotaEmpenhoService service;

    // 3. Testes
    @Test
    void deveEmitirEmpenhoQuandoSaldoSuficiente() {
        // Arrange
        when(dotacaoService.consultarSaldo(anyLong()))
            .thenReturn(SALDO_SUFICIENTE);

        // Act
        var resultado = service.emitir(requisicaoPadrao(VALOR_EMPENHO));

        // Assert
        assertThat(resultado.status()).isEqualTo(StatusNE.A_LIQUIDAR);
        verify(repositorio).save(any());
    }
}
```

### Testes de Integração (Testcontainers + PostgreSQL)

Testam Controllers + Services + Repositórios com banco PostgreSQL real em container.

**Casos obrigatórios por fluxo crítico:**

| Fluxo | Casos |
|---|---|
| Emissão NE | saldo suficiente; saldo insuficiente; dotação inativa; fornecedor inativo |
| Anulação NE | total sem liquidações; total com liquidação (deve falhar); parcial em ORDINARIO (deve falhar) |
| Reforço NE | mesmo exercício; exercício diferente (deve falhar) |
| Aprovação NC | saldo disponível; saldo insuficiente; NC já aprovada (deve falhar) |
| Estorno NC | créditos não empenhados; créditos já empenhados no destino (deve falhar) |
| Registro NL | dentro do limite; acima do empenhado (deve falhar); parcial em ORDINARIO (deve falhar) |
| Estorno NL | sem OB vinculada; com OB vinculada (deve falhar) |
| Emissão OB | liquidação registrada; liquidação estornada (deve falhar) |
| Login | credenciais válidas; credenciais inválidas; conta bloqueada |
| Token | geração; revogação; uso após revogação (deve falhar); token expirado (deve falhar) |

### Testes de Validação de API (MockMvc)

- Campos obrigatórios ausentes → 400 com detalhes
- Autenticação ausente → 401
- Token expirado ou revogado → 401
- Erros de negócio → 422 no formato RFC 7807

---

## Frontend

### Testes de Componente (Jest + React Testing Library)

```js
// Constantes no topo do arquivo de teste
const NUMERO_EMPENHO  = '2025MIN_ED000001';
const VALOR_EMPENHO   = 80_000;
const NOME_FORNECEDOR = 'Empresa ABC Ltda';
const ERRO_SALDO      = 'Saldo insuficiente';

describe('FormularioNotaEmpenho', () => {
    test('deve exibir saldo ao selecionar dotação', async () => { ... });
    test('deve desabilitar botão durante envio', async () => { ... });
    test('deve exibir erro de saldo insuficiente', async () => { ... });
    test('deve exibir modal ao confirmar anulação', async () => { ... });
});
```

---

## CI Pipeline

```yaml
jobs:
  backend:
    - setup-java 21
    - mvn verify                     # compila + testes + JaCoCo
    - falha se cobertura geral < 80%
    - falha se cobertura pacotes críticos < 90%

  frontend:
    - setup-node 22
    - npm ci && npm run lint
    - npm test -- --coverage
    - falha se cobertura < 70%
```

PR bloqueado para merge se qualquer job falhar.

---

## Processo de QA por Fase

Ver `13-qa-process.md` para o processo completo de validação e sign-off de cada fase.
