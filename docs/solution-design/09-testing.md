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
class CommitmentServiceTest {

    static final BigDecimal COMMITMENT_VALUE     = BigDecimal.valueOf(80_000);
    static final BigDecimal SUFFICIENT_BALANCE   = BigDecimal.valueOf(500_000);
    static final BigDecimal INSUFFICIENT_BALANCE = BigDecimal.valueOf(50_000);
    static final BigDecimal SUPPLEMENT_VALUE     = BigDecimal.valueOf(20_000);
    static final String     PROCESS_NUMBER       = "23000.001234/2025-01";
    static final String     OBJECT_DESCRIPTION   = "Aquisição de equipamentos de TI";
    static final Integer    CURRENT_FISCAL_YEAR  = 2025;
    static final Integer    PREVIOUS_FISCAL_YEAR = 2024;

    @Test
    void shouldIssueCommitmentWhenBalanceSufficient() {
        var dotacao = allotmentWithBalance(SUFFICIENT_BALANCE);
        var requisicao = commitmentRequest(COMMITMENT_VALUE, PROCESS_NUMBER);
        // ...
    }
}
```

**Java — errado:**
```java
@Test
void shouldIssueCommitment() {
    var dotacao = allotmentWithBalance(new BigDecimal("500000")); // ← magic number
    var requisicao = new CreateCommitmentRequest(80000.0, "23000.001234/2025-01"); // ← magic values
}
```

**JavaScript/TypeScript — correto:**
```js
const COMMITMENT_VALUE           = 80_000;
const SUFFICIENT_BALANCE         = 500_000;
const VENDOR_NAME                = 'Empresa ABC Ltda';
const VENDOR_CNPJ                = '12345678000190';
const INSUFFICIENT_BALANCE_MESSAGE = 'Saldo insuficiente';

test('deve exibir erro quando saldo for insuficiente', () => {
    // usa constantes acima
});
```

### Nomenclatura de Testes

Padrão: `should[Result]When[Condition]`

```java
// Backend
void shouldIssueCommitmentWhenBalanceSufficient()
void shouldThrowExceptionWhenBalanceInsufficient()
void shouldPreventPartialVoidingOfOrdinaryCommitment()
void shouldPreventSettlementReversalWithLinkedPaymentOrder()
void shouldLockAccountAfter5InvalidAttempts()

// Frontend
test('should display available balance when selecting allotment')
test('should disable Issue button during request')
test('should show confirmation modal when voiding commitment')
test('should clear form after successful issuance')
```

---

## Backend

### Testes Unitários (JUnit 5 + Mockito)

Testam a camada de Service em isolamento. Repositórios são mockados com Mockito.

**Estrutura de cada teste unitário:**
```java
class CommitmentServiceTest {

    // 1. Constantes no topo
    static final BigDecimal COMMITMENT_VALUE   = BigDecimal.valueOf(80_000);
    static final BigDecimal SUFFICIENT_BALANCE = BigDecimal.valueOf(500_000);

    // 2. Mocks e instância do service
    @Mock CommitmentRepository repositorio;
    @Mock AllotmentService allotmentService;
    @InjectMocks CommitmentService service;

    // 3. Testes
    @Test
    void shouldIssueCommitmentWhenBalanceSufficient() {
        // Arrange
        when(allotmentService.consultarSaldo(anyLong()))
            .thenReturn(SUFFICIENT_BALANCE);

        // Act
        var result = service.issue(defaultRequest(COMMITMENT_VALUE));

        // Assert
        assertThat(result.status()).isEqualTo(CommitmentStatus.PENDING_SETTLEMENT);
        verify(repositorio).save(any());
    }
}
```

### Testes de Integração (Testcontainers + PostgreSQL)

Testam Controllers + Services + Repositórios com banco PostgreSQL real em container.

**Casos obrigatórios por fluxo crítico:**

| Fluxo | Casos |
|---|---|
| Commitment issuance | sufficient balance; insufficient balance; inactive allotment; inactive vendor |
| Commitment voiding | total without settlements; total with settlement (must fail); partial on ORDINARIO (must fail) |
| Commitment reinforcement | same fiscal year; different fiscal year (must fail) |
| Credit note approval | available balance; insufficient balance; already approved (must fail) |
| Credit note reversal | credits not committed; credits already committed at destination (must fail) |
| Settlement registration | within limit; above committed (must fail); partial on ORDINARIO (must fail) |
| Settlement reversal | without linked payment order; with linked payment order (must fail) |
| Payment order issuance | registered settlement; reversed settlement (must fail) |
| Login | valid credentials; invalid credentials; locked account |
| Token | generation; revocation; use after revocation (must fail); expired token (must fail) |

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
const COMMITMENT_NUMBER = '2025MIN_ED000001';
const COMMITMENT_VALUE  = 80_000;
const VENDOR_NAME       = 'Empresa ABC Ltda';
const ERRO_SALDO        = 'Saldo insuficiente';

describe('CommitmentForm', () => {
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
