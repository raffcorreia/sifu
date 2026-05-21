# 05 — Backend Design

> **Layer:** Core Design | **Depends on:** 03-architecture, 04-database-design

## Architectural Principles

### Hexagonal Architecture (Ports & Adapters)

The single governing rule: **all dependencies point inward**. The domain layer has zero framework dependencies — no Spring, no JPA, no Hibernate.

```
┌─────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE                        │
│  ┌───────────────────────────────────────────────────┐  │
│  │                  APPLICATION                       │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │                  DOMAIN                      │  │  │
│  │  │   model/  ·  port/in  ·  port/out            │  │  │
│  │  │   (pure Java — zero framework deps)          │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │   usecase/ (implements port/in, uses port/out)     │  │
│  └───────────────────────────────────────────────────┘  │
│   web/  persistence/  security/  audit/  notification/  │
└─────────────────────────────────────────────────────────┘
```

Adapters (infrastructure) call into ports. Ports are owned by the domain. Nothing inside the hexagon knows the adapters exist.

### SOLID Applied

| Principle | Application |
|---|---|
| **S** — Single Responsibility | Each use case class handles one operation; controllers only serialize/deserialize |
| **O** — Open/Closed | New operations = new use case classes; existing ones untouched |
| **L** — Liskov Substitution | Repository ports have one implementation (JPA); tests substitute with Mockito |
| **I** — Interface Segregation | One interface per use case (`IssueCommitmentUseCase`, not `CommitmentService`) |
| **D** — Dependency Inversion | Application depends on domain ports; infrastructure implements them |

### Clean Code Conventions

- **Constructor injection only** — never `@Autowired` on fields
- **Guard clauses** — fail fast; error checks first, happy path last
- **No magic values** — named constants or enums
- **Small, focused methods** — one level of abstraction per method
- **`Optional<T>`** — no null returns from repository ports
- **Java Records** — for immutable DTOs and Value Objects
- **No explanatory comments** — code is self-documenting; comments only for non-obvious WHY

---

## Package Structure

```
br.gov.sifu/
│
├── domain/                                  ← ZERO framework dependencies
│   ├── model/
│   │   ├── Commitment.java
│   │   ├── CreditNote.java
│   │   ├── BudgetAllotment.java
│   │   ├── Settlement.java
│   │   ├── PaymentOrder.java
│   │   ├── ManagingUnit.java
│   │   ├── Vendor.java
│   │   ├── User.java
│   │   ├── IntegrationToken.java
│   │   ├── AuditEntry.java
│   │   └── vo/
│   │       ├── Money.java                   ← Value Object (record)
│   │       ├── DocumentNumber.java          ← Value Object (record)
│   │       ├── CommitmentId.java
│   │       ├── CommitmentStatus.java        ← enum
│   │       ├── CommitmentType.java          ← enum
│   │       └── DocumentType.java            ← enum
│   ├── port/
│   │   ├── in/                              ← Use case interfaces (Input Ports)
│   │   │   ├── IssueCommitmentUseCase.java
│   │   │   ├── VoidCommitmentUseCase.java
│   │   │   ├── ApproveCreditNoteUseCase.java
│   │   │   ├── CancelCreditNoteUseCase.java
│   │   │   ├── RegisterSettlementUseCase.java
│   │   │   ├── ProcessPaymentOrderUseCase.java
│   │   │   └── ...
│   │   └── out/                             ← Repository interfaces (Output Ports)
│   │       ├── CommitmentRepository.java
│   │       ├── AllotmentRepository.java
│   │       ├── CreditNoteRepository.java
│   │       ├── SettlementRepository.java
│   │       ├── PaymentOrderRepository.java
│   │       ├── ManagingUnitRepository.java
│   │       ├── VendorRepository.java
│   │       ├── UserRepository.java
│   │       ├── IntegrationTokenRepository.java
│   │       ├── AuditRepository.java
│   │       ├── DocumentSequenceRepository.java
│   │       └── NotificationPort.java
│   └── exception/                           ← Pure Java exceptions
│       ├── DomainException.java             ← base (extends RuntimeException)
│       ├── InsufficientBalanceException.java
│       ├── InvalidStateTransitionException.java
│       ├── PartialVoidingNotAllowedException.java
│       ├── EntityNotFoundException.java
│       └── VendorInactiveException.java
│
├── application/
│   └── usecase/                             ← @UseCase implementations
│       ├── IssueCommitmentService.java
│       ├── VoidCommitmentService.java
│       ├── ApproveCreditNoteService.java
│       ├── CancelCreditNoteService.java
│       ├── RegisterSettlementService.java
│       ├── ProcessPaymentOrderService.java
│       └── ...
│
└── infrastructure/
    ├── web/                                 ← HTTP adapters
    │   ├── CommitmentController.java
    │   ├── CreditNoteController.java
    │   ├── AllotmentController.java
    │   ├── SettlementController.java
    │   ├── PaymentOrderController.java
    │   ├── ManagingUnitController.java
    │   ├── VendorController.java
    │   ├── UserController.java
    │   ├── TokenController.java
    │   ├── ReportController.java
    │   ├── AuditController.java
    │   ├── AuthController.java
    │   ├── dto/                             ← Java Records only
    │   └── mapper/                          ← MapStruct interfaces
    ├── persistence/                         ← JPA adapters
    │   ├── entity/                          ← @Entity classes live here only
    │   │   ├── CommitmentJpaEntity.java
    │   │   ├── CreditNoteJpaEntity.java
    │   │   └── ...
    │   ├── repository/                      ← Spring Data JPA interfaces
    │   │   ├── CommitmentJpaRepository.java
    │   │   └── ...
    │   └── adapter/                         ← implement domain port/out
    │       ├── CommitmentPersistenceAdapter.java
    │       └── ...
    ├── security/
    │   ├── SecurityConfig.java
    │   ├── JwtFilter.java
    │   └── JwtService.java
    ├── notification/
    │   └── EmailNotificationAdapter.java    ← implements NotificationPort
    ├── audit/
    │   └── AuditInterceptor.java            ← @Aspect
    └── config/
        └── ApplicationConfig.java           ← @Bean wiring
```

---

## Domain Layer

### Rich Domain Model

Domain objects own their business rules. State transitions, invariants, and validations live in the model — not in services.

```java
// domain/model/Commitment.java
public class Commitment {

    private CommitmentId id;
    private DocumentNumber number;
    private Money value;
    private CommitmentStatus status;
    private ManagingUnit managingUnit;
    private BudgetAllotment allotment;
    private Vendor vendor;
    private int fiscalYear;

    // Factory method — the only way to create a new Commitment
    public static Commitment issue(
            DocumentNumber number,
            Money value,
            ManagingUnit managingUnit,
            BudgetAllotment allotment,
            Vendor vendor,
            int fiscalYear) {
        if (!vendor.isActive()) {
            throw new VendorInactiveException(vendor.getId());
        }
        if (allotment.availableBalance().isLessThan(value)) {
            throw new InsufficientBalanceException(allotment.getId(), allotment.availableBalance(), value);
        }
        var commitment = new Commitment();
        commitment.number = number;
        commitment.value = value;
        commitment.status = CommitmentStatus.PENDING_SETTLEMENT;
        commitment.managingUnit = managingUnit;
        commitment.allotment = allotment;
        commitment.vendor = vendor;
        commitment.fiscalYear = fiscalYear;
        allotment.reserveBalance(value);
        return commitment;
    }

    // Reconstitution — used by persistence adapters only
    public static Commitment reconstitute(/* all fields */) { ... }

    public void voidCommitment() {
        if (status != CommitmentStatus.PENDING_SETTLEMENT) {
            throw new InvalidStateTransitionException("Commitment", status, CommitmentStatus.VOIDED);
        }
        this.status = CommitmentStatus.VOIDED;
        this.allotment.releaseBalance(this.value);
    }

    public void markAsSettled(Money settledAmount) {
        if (settledAmount.equals(this.value)) {
            this.status = CommitmentStatus.SETTLED;
        } else {
            this.status = CommitmentStatus.PARTIALLY_SETTLED;
        }
    }
}
```

### Value Objects

```java
// domain/model/vo/Money.java
public record Money(BigDecimal amount) {

    public Money {
        Objects.requireNonNull(amount, "amount must not be null");
        if (amount.scale() > 2) {
            throw new DomainException("Money allows at most 2 decimal places");
        }
    }

    public Money add(Money other)      { return new Money(amount.add(other.amount)); }
    public Money subtract(Money other) { return new Money(amount.subtract(other.amount)); }
    public boolean isLessThan(Money other)        { return amount.compareTo(other.amount) < 0; }
    public boolean isNegativeOrZero()             { return amount.compareTo(BigDecimal.ZERO) <= 0; }
    public static Money of(BigDecimal amount)     { return new Money(amount); }
    public static Money of(String amount)         { return new Money(new BigDecimal(amount)); }
}
```

```java
// domain/model/vo/DocumentNumber.java
public record DocumentNumber(String value) {

    private static final Pattern FORMAT = Pattern.compile("\\d{4}[A-Z0-9_]{1,20}\\d{6}");

    public DocumentNumber {
        Objects.requireNonNull(value, "value must not be null");
        if (!FORMAT.matcher(value).matches()) {
            throw new DomainException("Invalid document number format: " + value);
        }
    }
}
```

### Input Ports (Use Case Interfaces)

Each use case is a single-method interface. Commands are Java Records.

```java
// domain/port/in/IssueCommitmentUseCase.java
public interface IssueCommitmentUseCase {
    Commitment issue(IssueCommitmentCommand command);
}
```

```java
// domain/port/in/IssueCommitmentCommand.java
public record IssueCommitmentCommand(
    Long allotmentId,
    Long vendorId,
    Long managingUnitId,
    BigDecimal value,
    CommitmentType type,
    String description,
    int fiscalYear
) {}
```

### Output Ports (Repository Interfaces)

```java
// domain/port/out/CommitmentRepository.java
public interface CommitmentRepository {
    Commitment save(Commitment commitment);
    Optional<Commitment> findById(CommitmentId id);
    List<Commitment> findByManagingUnit(Long managingUnitId, int fiscalYear);
}
```

---

## Application Layer

### `@UseCase` Annotation

```java
// application/usecase/UseCase.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Service
public @interface UseCase {}
```

### Use Case Implementation

```java
// application/usecase/IssueCommitmentService.java
@UseCase
@Transactional
public class IssueCommitmentService implements IssueCommitmentUseCase {

    private final CommitmentRepository commitmentRepository;
    private final AllotmentRepository allotmentRepository;
    private final VendorRepository vendorRepository;
    private final ManagingUnitRepository managingUnitRepository;
    private final DocumentSequenceRepository sequenceRepository;

    public IssueCommitmentService(
            CommitmentRepository commitmentRepository,
            AllotmentRepository allotmentRepository,
            VendorRepository vendorRepository,
            ManagingUnitRepository managingUnitRepository,
            DocumentSequenceRepository sequenceRepository) {
        this.commitmentRepository = commitmentRepository;
        this.allotmentRepository = allotmentRepository;
        this.vendorRepository = vendorRepository;
        this.managingUnitRepository = managingUnitRepository;
        this.sequenceRepository = sequenceRepository;
    }

    @Override
    @Auditable(operation = "ISSUE_COMMITMENT", entity = "Commitment")
    public Commitment issue(IssueCommitmentCommand command) {
        var allotment = allotmentRepository.findById(command.allotmentId())
            .orElseThrow(() -> new EntityNotFoundException("BudgetAllotment", command.allotmentId()));
        var vendor = vendorRepository.findById(command.vendorId())
            .orElseThrow(() -> new EntityNotFoundException("Vendor", command.vendorId()));
        var managingUnit = managingUnitRepository.findById(command.managingUnitId())
            .orElseThrow(() -> new EntityNotFoundException("ManagingUnit", command.managingUnitId()));

        var number = sequenceRepository.nextNumber(DocumentType.COMMITMENT, command.fiscalYear());
        var commitment = Commitment.issue(
            number,
            Money.of(command.value()),
            managingUnit,
            allotment,
            vendor,
            command.fiscalYear()
        );
        return commitmentRepository.save(commitment);
    }
}
```

**Guard clause pattern** — all error checks first, happy path at the end. No nested conditionals.

---

## Infrastructure Layer

### Controllers (Thin Adapters)

Controllers do exactly three things: deserialize, delegate, serialize.

```java
// infrastructure/web/CommitmentController.java
@RestController
@RequestMapping("/api/v1/commitments")
@RequiredArgsConstructor
public class CommitmentController {

    private final IssueCommitmentUseCase issueCommitmentUseCase;
    private final CommitmentMapper commitmentMapper;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public CommitmentResponse issue(@RequestBody @Valid IssueCommitmentRequest request) {
        var command = commitmentMapper.toCommand(request);
        var commitment = issueCommitmentUseCase.issue(command);
        return commitmentMapper.toResponse(commitment);
    }
}
```

### DTOs as Java Records

```java
// infrastructure/web/dto/IssueCommitmentRequest.java
public record IssueCommitmentRequest(
    @NotNull Long allotmentId,
    @NotNull Long vendorId,
    @NotNull Long managingUnitId,
    @NotNull @Positive BigDecimal value,
    @NotNull CommitmentType type,
    @NotBlank String description,
    @NotNull @Min(2000) Integer fiscalYear
) {}
```

```java
// infrastructure/web/dto/CommitmentResponse.java
public record CommitmentResponse(
    Long id,
    String number,
    BigDecimal value,
    CommitmentStatus status,
    CommitmentType type,
    String description,
    Long allotmentId,
    Long vendorId,
    Long managingUnitId,
    int fiscalYear,
    Instant createdAt
) {}
```

### Persistence Adapters

`@Entity` annotations live **only** in `infrastructure/persistence/entity/`. Domain model objects are pure Java.

```java
// infrastructure/persistence/adapter/CommitmentPersistenceAdapter.java
@Component
@RequiredArgsConstructor
public class CommitmentPersistenceAdapter implements CommitmentRepository {

    private final CommitmentJpaRepository jpaRepository;
    private final CommitmentPersistenceMapper mapper;

    @Override
    public Commitment save(Commitment commitment) {
        var entity = mapper.toEntity(commitment);
        var saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Commitment> findById(CommitmentId id) {
        return jpaRepository.findById(id.value()).map(mapper::toDomain);
    }
}
```

### ApplicationConfig — Explicit Wiring

```java
// infrastructure/config/ApplicationConfig.java
@Configuration
public class ApplicationConfig {

    @Bean
    public IssueCommitmentUseCase issueCommitmentUseCase(
            CommitmentRepository commitmentRepository,
            AllotmentRepository allotmentRepository,
            VendorRepository vendorRepository,
            ManagingUnitRepository managingUnitRepository,
            DocumentSequenceRepository sequenceRepository) {
        return new IssueCommitmentService(
            commitmentRepository, allotmentRepository,
            vendorRepository, managingUnitRepository, sequenceRepository);
    }
}
```

### Pagination

```java
// infrastructure/web/dto/PageResponse.java
public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean last
) {
    public static <T> PageResponse<T> of(Page<T> page) {
        return new PageResponse<>(
            page.getContent(),
            page.getNumber(),
            page.getSize(),
            page.getTotalElements(),
            page.getTotalPages(),
            page.isLast()
        );
    }
}
```

All list endpoints accept:

| Parameter | Default | Maximum |
|---|---|---|
| `page` | 0 | — |
| `size` | 20 | 100 |
| `sortBy` | endpoint-specific | — |
| `direction` | `ASC` | `ASC` or `DESC` |

---

## Error Handling

All error responses follow RFC 7807 (Problem Details):

```json
{
  "type": "https://sifu.gov.br/errors/insufficient-balance",
  "title": "Insufficient balance",
  "status": 422,
  "detail": "Available balance for allotment 42 is R$ 1,000.00; requested: R$ 1,500.00",
  "instance": "/api/v1/commitments"
}
```

| Exception | HTTP Status |
|---|---|
| `EntityNotFoundException` | 404 |
| `InsufficientBalanceException` | 422 |
| `InvalidStateTransitionException` | 422 |
| `VendorInactiveException` | 422 |
| `PartialVoidingNotAllowedException` | 422 |
| Bean Validation (`@Valid`) | 400 |
| Unauthenticated | 401 |
| Token expired / revoked | 401 |
| Account locked | 423 |

---

## Audit

`@Auditable` is a custom annotation processed by `AuditInterceptor` (Spring AOP aspect):

```java
@Auditable(operation = "ISSUE_COMMITMENT", entity = "Commitment")
public Commitment issue(IssueCommitmentCommand command) { ... }
```

The interceptor captures automatically: logged-in user, request IP, before/after state (JSON), timestamp.

---

## Security

```
Public endpoints:      POST /api/v1/auth/login
                       POST /api/v1/auth/recover-password
                       GET  /swagger-ui/**
                       GET  /api-docs/**
                       GET  /actuator/health

Authenticated:         all other /api/v1/**
```

`JwtFilter` accepts two token types:
- **Session JWT** — issued on login, expires in 8h, claims: `sub` (login), `type=SESSION`
- **Integration token** — issued via `/api/v1/tokens`, configurable expiry, claims: `type=INTEGRATION`

---

## Architecture Tests (ArchUnit)

```java
// HexagonalArchitectureTest.java
@AnalyzeClasses(packages = "br.gov.sifu")
class HexagonalArchitectureTest {

    @ArchTest
    static final ArchRule domainHasNoFrameworkDependencies =
        noClasses().that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
            .resideInAnyPackage(
                "org.springframework..",
                "jakarta.persistence..",
                "org.hibernate..");

    @ArchTest
    static final ArchRule applicationDoesNotDependOnInfrastructure =
        noClasses().that().resideInAPackage("..application..")
            .should().dependOnClassesThat()
            .resideInAPackage("..infrastructure..");

    @ArchTest
    static final ArchRule controllersAreOnlyInWeb =
        classes().that().haveSimpleNameEndingWith("Controller")
            .should().resideInAPackage("..infrastructure.web..");

    @ArchTest
    static final ArchRule entitiesAreOnlyInPersistence =
        classes().that().areAnnotatedWith(Entity.class)
            .should().resideInAPackage("..infrastructure.persistence.entity..");

    @ArchTest
    static final ArchRule useCasesImplementTheirPort =
        classes().that().areAnnotatedWith(UseCase.class)
            .should().implement(
                assignableTo(resideInAPackage("..domain.port.in..")));
}
```

---

## Testing Strategy by Layer

| Layer | Test Type | Tools | Spring Context |
|---|---|---|---|
| `domain/` | Unit | JUnit 5 | None |
| `application/usecase/` | Unit | JUnit 5 + Mockito | None |
| `infrastructure/persistence/` | Integration | `@DataJpaTest` + Testcontainers | Partial |
| `infrastructure/web/` | Slice | `@WebMvcTest` + Mockito | Partial |
| Full stack | Integration | `@SpringBootTest` + Testcontainers | Full |
| Architecture | Architecture | ArchUnit | None |

**Domain tests** are the fastest and most valuable — pure Java, zero Spring, run in milliseconds.

```java
// CommitmentTest.java (domain layer — pure JUnit, no Spring)
class CommitmentTest {

    @Test
    void shouldThrowWhenVendorIsInactive() {
        var vendor = Vendor.reconstitute(1L, "ACME", false);
        var allotment = allotmentWithBalance(Money.of("5000.00"));

        assertThatThrownBy(() -> Commitment.issue(
            DocumentNumber.of("2025NE000001"),
            Money.of("1000.00"),
            managingUnit(), allotment, vendor, 2025))
            .isInstanceOf(VendorInactiveException.class);
    }

    @Test
    void shouldReserveAllotmentBalanceOnIssue() {
        var allotment = allotmentWithBalance(Money.of("5000.00"));
        var vendor = Vendor.reconstitute(1L, "ACME", true);

        Commitment.issue(documentNumber(), Money.of("2000.00"),
            managingUnit(), allotment, vendor, 2025);

        assertThat(allotment.availableBalance()).isEqualTo(Money.of("3000.00"));
    }
}
```
