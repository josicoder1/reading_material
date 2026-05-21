
# 🎓 Advanced Domain-Driven Design & Distributed Systems Masterclass

## Course Overview
This course covers tactical DDD patterns, hexagonal architecture, and practical distributed systems implementation with Kafka and the Outbox pattern.

---

# MODULE 1: DDD Tactical Design Patterns

## 1.1 Aggregate

### Definition
An **Aggregate** is a cluster of associated objects that we treat as a single unit for data changes. Each aggregate has a **root entity** (Aggregate Root) and a boundary that defines what is included inside the aggregate.

### Key Characteristics

| Characteristic | Description |
|---------------|-------------|
| **Boundary** | Defines consistency boundary - everything inside is strongly consistent |
| **Root Entity** | The only member external objects can hold references to |
| **Transactional Consistency** | All changes within an aggregate are committed atomically |
| **Invariants** | Business rules that must always be true within the aggregate |
| **Small & Focused** | Should contain only what is necessary for the business operation |

### Example: Order Aggregate

```java
public class Order { // Aggregate Root
    private OrderId id;
    private CustomerId customerId; // Reference to another aggregate (by ID only!)
    private List<OrderLine> orderLines; // Value objects or entities inside boundary
    private Money totalAmount;
    private OrderStatus status;
    private List<DomainEvent> domainEvents; // Events to publish

    // Constructor - enforces invariants at creation
    public Order(CustomerId customerId, List<OrderLine> lines) {
        this.id = OrderId.generate();
        this.customerId = customerId;
        this.orderLines = new ArrayList<>(lines);
        this.status = OrderStatus.PENDING;
        this.totalAmount = calculateTotal();

        // Validate invariants
        if (orderLines.isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one line");
        }

        // Register domain event
        registerEvent(new OrderCreatedEvent(this.id, this.customerId, this.totalAmount));
    }

    // Business method - encapsulates business logic
    public void addLine(ProductId productId, Quantity quantity, Money price) {
        // Check business rules
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot modify submitted order");
        }

        OrderLine line = new OrderLine(productId, quantity, price);
        orderLines.add(line);
        this.totalAmount = calculateTotal();

        registerEvent(new OrderLineAddedEvent(this.id, productId, quantity));
    }

    public void submit() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Order already submitted");
        }
        if (totalAmount.isLessThan(Money.of(10))) { // Business invariant
            throw new IllegalArgumentException("Minimum order amount is $10");
        }

        this.status = OrderStatus.SUBMITTED;
        registerEvent(new OrderSubmittedEvent(this.id, this.totalAmount));
    }

    // Only expose ID references to other aggregates
    public CustomerId getCustomerId() { return customerId; }

    // Never expose mutable internal state directly
    public List<OrderLine> getOrderLines() { 
        return Collections.unmodifiableList(orderLines); 
    }

    private void registerEvent(DomainEvent event) {
        domainEvents.add(event);
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        domainEvents.clear();
    }
}
```

### Aggregate Design Rules (CRITICAL)

**Rule 1: Reference Other Aggregates by Identity Only**
```java
// ❌ WRONG - Holding reference to another aggregate
public class Order {
    private Customer customer; // Don't do this!
}

// ✅ CORRECT - Reference by ID
public class Order {
    private CustomerId customerId; // Just the identity
}
```

**Rule 2: One Transaction = One Aggregate**
```java
// ❌ WRONG - Modifying two aggregates in one transaction
@Transactional
public void transferMoney(AccountId from, AccountId to, Money amount) {
    Account fromAccount = accountRepo.findById(from);
    Account toAccount = accountRepo.findById(to); // Different aggregate!

    fromAccount.debit(amount);
    toAccount.credit(amount); // ❌ Don't modify multiple aggregates!
}

// ✅ CORRECT - Use domain events for cross-aggregate communication
@Transactional
public void transferMoney(AccountId from, AccountId to, Money amount) {
    Account fromAccount = accountRepo.findById(from);
    fromAccount.debit(amount); // Only modify one aggregate
    fromAccount.registerEvent(new MoneyTransferInitiatedEvent(from, to, amount));
    accountRepo.save(fromAccount);
    // Event handler will credit the other account asynchronously
}
```

**Rule 3: Keep Aggregates Small**
```java
// ❌ WRONG - Huge aggregate with everything
public class Customer {
    private List<Order> orders; // Don't embed orders!
    private List<Address> addresses;
    private List<PaymentMethod> paymentMethods;
    private LoyaltyPoints points;
    private Preferences preferences;
    // ... 20 more fields
}

// ✅ CORRECT - Focused aggregate
public class Customer {
    private CustomerId id;
    private String name;
    private Email email;
    private List<Address> addresses; // Only closely related data

    // Orders are a separate aggregate!
}
```

---

## 1.2 Entity

### Definition
An **Entity** is an object that has a distinct identity that runs through time and different states. Two entities with the same identity are considered the same even if their attributes differ.

### Characteristics
- Has a unique identifier (ID)
- Identity remains constant throughout its lifecycle
- Mutable - attributes can change
- Equality based on identity, not attributes

### Example: Customer Entity

```java
public class Customer {
    private final CustomerId id; // Identity - never changes
    private String name;
    private Email email;
    private CustomerStatus status;
    private LocalDateTime createdAt;

    // Entity equality based on ID
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Customer customer = (Customer) o;
        return id.equals(customer.id); // Only compare IDs!
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

    // Business methods that change state
    public void changeEmail(Email newEmail) {
        if (this.status == CustomerStatus.BLOCKED) {
            throw new IllegalStateException("Blocked customers cannot change email");
        }
        this.email = newEmail;
        registerEvent(new CustomerEmailChangedEvent(this.id, newEmail));
    }

    public void upgradeToPremium() {
        if (this.status != CustomerStatus.ACTIVE) {
            throw new IllegalStateException("Only active customers can upgrade");
        }
        this.status = CustomerStatus.PREMIUM;
        registerEvent(new CustomerUpgradedEvent(this.id));
    }
}
```

### Entity vs Value Object Decision Tree

```
Does the object need identity?
│
├── YES → Entity
│   ├── Has lifecycle (created, modified, deleted)
│   ├── Tracked by ID in database
│   └── Equality by ID
│
└── NO → Value Object
    ├── Immutable (preferred)
    ├── Equality by all attributes
    └── Can be freely replaced/duplicated
```

---

## 1.3 Value Object

### Definition
A **Value Object** is an immutable object that describes some characteristic or attribute but has no conceptual identity. Value objects are defined entirely by their attributes.

### Characteristics
- **Immutable** - cannot be changed after creation
- **No identity** - equality based on attribute values
- **Replaceable** - if you need to change it, create a new one
- **Self-validating** - validates invariants at construction

### Example: Money Value Object

```java
public final class Money { // final class - cannot be subclassed
    private final BigDecimal amount;
    private final Currency currency;

    private Money(BigDecimal amount, Currency currency) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount must be non-negative");
        }
        if (currency == null) {
            throw new IllegalArgumentException("Currency is required");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency;
    }

    public static Money of(BigDecimal amount, Currency currency) {
        return new Money(amount, currency);
    }

    public static Money zero(Currency currency) {
        return new Money(BigDecimal.ZERO, currency);
    }

    // Operations return NEW instances (immutability)
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot subtract different currencies");
        }
        BigDecimal result = this.amount.subtract(other.amount);
        if (result.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Result cannot be negative");
        }
        return new Money(result, this.currency);
    }

    public Money multiply(int factor) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(factor)), this.currency);
    }

    public boolean isGreaterThan(Money other) {
        return this.amount.compareTo(other.amount) > 0;
    }

    // No setters! Immutability enforced
    public BigDecimal getAmount() { return amount; }
    public Currency getCurrency() { return currency; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return amount.equals(money.amount) && currency.equals(money.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }

    @Override
    public String toString() {
        return currency.getSymbol() + amount;
    }
}
```

### More Value Object Examples

```java
// Email Value Object
public final class Email {
    private final String value;

    public Email(String value) {
        if (value == null || !value.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        this.value = value.toLowerCase();
    }
}

// Address Value Object
public final class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    private final Country country;

    public Address(String street, String city, String zipCode, Country country) {
        // Validation logic
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
        this.country = country;
    }

    // Can create variants without modifying original
    public Address withStreet(String newStreet) {
        return new Address(newStreet, this.city, this.zipCode, this.country);
    }
}

// Quantity Value Object
public final class Quantity {
    private final int value;

    public Quantity(int value) {
        if (value <= 0) throw new IllegalArgumentException("Quantity must be positive");
        if (value > 1000) throw new IllegalArgumentException("Max quantity is 1000");
        this.value = value;
    }

    public Quantity add(Quantity other) {
        return new Quantity(this.value + other.value);
    }
}
```

### Why Value Objects Are Powerful

1. **Validation at construction** - Invalid states are impossible
2. **Thread-safe** - Immutability = no synchronization needed
3. **Safe to share** - Can be passed around without defensive copies
4. **Explicit business concepts** - `Money` is clearer than `BigDecimal`
5. **Behavior rich** - Business logic lives with the data

---

## 1.4 Domain Event

### Definition
A **Domain Event** is an event that captures the occurrence of something that domain experts care about. It represents a fact that has happened in the past and is immutable.

### Characteristics
- Named in past tense (`OrderCreated`, `PaymentReceived`)
- Immutable - represents something that already happened
- Contains relevant data about what occurred
- Enables loose coupling between aggregates and bounded contexts

### Example: Domain Events

```java
// Base interface
public interface DomainEvent {
    EventId getEventId();
    Instant getOccurredOn();
}

// Concrete event
public class OrderCreatedEvent implements DomainEvent {
    private final EventId eventId;
    private final Instant occurredOn;
    private final OrderId orderId;
    private final CustomerId customerId;
    private final Money totalAmount;

    public OrderCreatedEvent(OrderId orderId, CustomerId customerId, Money totalAmount) {
        this.eventId = EventId.generate();
        this.occurredOn = Instant.now();
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
    }

    // Getters only - no setters (immutable)
    @Override
    public EventId getEventId() { return eventId; }

    @Override
    public Instant getOccurredOn() { return occurredOn; }

    public OrderId getOrderId() { return orderId; }
    public CustomerId getCustomerId() { return customerId; }
    public Money getTotalAmount() { return totalAmount; }
}

// Another event
public class PaymentReceivedEvent implements DomainEvent {
    private final EventId eventId;
    private final Instant occurredOn;
    private final PaymentId paymentId;
    private final OrderId orderId;
    private final Money amount;
    private final PaymentMethod method;

    // ... constructor and getters
}
```

### Event Publishing Inside Aggregate

```java
public class Order {
    private List<DomainEvent> domainEvents = new ArrayList<>();

    public void confirmPayment(PaymentId paymentId, Money amount) {
        if (status != OrderStatus.SUBMITTED) {
            throw new IllegalStateException("Order must be submitted");
        }

        this.status = OrderStatus.PAID;
        this.paymentId = paymentId;

        // Register event - don't publish directly to Kafka!
        registerEvent(new OrderPaidEvent(
            this.id, 
            paymentId, 
            amount, 
            Instant.now()
        ));
    }

    private void registerEvent(DomainEvent event) {
        domainEvents.add(event);
    }

    // Called by repository or application service
    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        domainEvents.clear();
    }
}
```

---

# MODULE 2: Hexagonal Architecture (Ports & Adapters)

## 2.1 Core Concept

Hexagonal Architecture (by Alistair Cockburn) separates the application into:
- **Inside**: Domain logic (pure, framework-independent)
- **Outside**: Infrastructure (databases, web frameworks, message queues)
- **Ports**: Interfaces that define how the outside world interacts with the application
- **Adapters**: Concrete implementations of ports

```
                    ┌─────────────────┐
                    │   REST API      │
                    │   Adapter       │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         │    ┌──────────────┴──────────────┐    │
         │    │      PRIMARY/DRIVING        │    │
         │    │          PORTS              │    │
         │    │  (Inbound interfaces)       │    │
         │    └──────────────┬──────────────┘    │
         │                   │                   │
         │         ┌─────────┴─────────┐         │
         │         │   APPLICATION     │         │
         │         │     LAYER         │         │
         │         │  (Use Cases)      │         │
         │         └─────────┬─────────┘         │
         │                   │                   │
         │         ┌─────────┴─────────┐         │
         │         │   DOMAIN LAYER    │         │
         │         │ (Aggregates, VO,  │         │
         │         │  Domain Services) │         │
         │         └─────────┬─────────┘         │
         │                   │                   │
         │    ┌──────────────┴──────────────┐    │
         │    │     SECONDARY/DRIVEN        │    │
         │    │          PORTS              │    │
         │    │  (Outbound interfaces)      │    │
         │    └──────────────┬──────────────┘    │
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────┴────┐      ┌──────┴──────┐      ┌─────┴─────┐
    │  JPA    │      │   Kafka     │      │  External │
    │Adapter  │      │   Adapter   │      │   API     │
    └─────────┘      └─────────────┘      │  Adapter  │
                                          └───────────┘
```

## 2.2 Ports (Interfaces)

### Primary/Driving Ports (Inbound)
What the application offers to the outside world:

```java
// Application Service Interface (Primary Port)
public interface OrderService {
    OrderId createOrder(CreateOrderCommand command);
    void submitOrder(OrderId orderId);
    void cancelOrder(OrderId orderId);
    OrderDto getOrder(OrderId orderId);
}

// Query Service (CQRS - separate read model)
public interface OrderQueryService {
    OrderSummary getOrderSummary(OrderId orderId);
    List<OrderSummary> findOrdersByCustomer(CustomerId customerId);
}
```

### Secondary/Driven Ports (Outbound)
What the application needs from the outside world:

```java
// Repository Port - abstracts persistence
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId id);
    List<Order> findByCustomerId(CustomerId customerId);
}

// Event Publisher Port - abstracts messaging
public interface DomainEventPublisher {
    void publish(List<DomainEvent> events);
}

// External Service Port - abstracts 3rd party APIs
public interface PaymentGateway {
    PaymentResult processPayment(PaymentRequest request);
}

// Notification Port
public interface NotificationService {
    void sendOrderConfirmation(CustomerId customerId, OrderId orderId);
}
```

## 2.3 Adapters (Implementations)

### Primary Adapters (Call the Application)

```java
// REST Controller Adapter
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService; // Primary port
    private final OrderQueryService queryService;

    @PostMapping
    public ResponseEntity<OrderId> createOrder(@RequestBody CreateOrderRequest request) {
        CreateOrderCommand command = mapToCommand(request);
        OrderId orderId = orderService.createOrder(command);
        return ResponseEntity.status(HttpStatus.CREATED).body(orderId);
    }

    @GetMapping("/{orderId}")
    public ResponseEntity<OrderSummary> getOrder(@PathVariable String orderId) {
        OrderSummary summary = queryService.getOrderSummary(new OrderId(orderId));
        return ResponseEntity.ok(summary);
    }
}

// Message Consumer Adapter (Kafka)
@Component
public class PaymentEventConsumer {

    private final OrderService orderService; // Primary port

    @KafkaListener(topics = "payments", groupId = "order-service")
    public void onPaymentReceived(PaymentReceivedEvent event) {
        orderService.confirmPayment(event.getOrderId(), event.getPaymentId());
    }
}
```

### Secondary Adapters (Called by the Application)

```java
// JPA Repository Adapter
@Repository
public class JpaOrderRepositoryAdapter implements OrderRepository { // Implements secondary port

    private final JpaOrderEntityRepository jpaRepository;
    private final OrderMapper mapper;

    @Override
    public Order save(Order order) {
        OrderEntity entity = mapper.toEntity(order);
        OrderEntity saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Order> findById(OrderId id) {
        return jpaRepository.findById(id.getValue())
            .map(mapper::toDomain);
    }
}

// Kafka Event Publisher Adapter
@Component
public class KafkaDomainEventPublisher implements DomainEventPublisher {

    private final KafkaTemplate<String, DomainEvent> kafkaTemplate;
    private final ObjectMapper objectMapper;

    @Override
    public void publish(List<DomainEvent> events) {
        for (DomainEvent event : events) {
            String topic = resolveTopic(event);
            String key = resolveKey(event);

            kafkaTemplate.send(topic, key, event)
                .whenComplete((result, ex) -> {
                    if (ex != null) {
                        log.error("Failed to publish event: {}", event.getEventId(), ex);
                    }
                });
        }
    }

    private String resolveTopic(DomainEvent event) {
        return event.getClass().getSimpleName().toLowerCase() + "-events";
    }
}

// Stripe Payment Gateway Adapter
@Component
public class StripePaymentGatewayAdapter implements PaymentGateway {

    private final StripeClient stripeClient;

    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        try {
            PaymentIntent intent = stripeClient.paymentIntents().create(
                PaymentIntentCreateParams.builder()
                    .setAmount(request.getAmountInCents())
                    .setCurrency(request.getCurrency())
                    .build()
            );
            return PaymentResult.success(intent.getId());
        } catch (StripeException e) {
            return PaymentResult.failure(e.getMessage());
        }
    }
}
```

## 2.4 Dependency Rule

**CRITICAL**: Dependencies point INWARD only!

```
Domain Layer (center)
    ↑ (depends on)
Application Layer
    ↑ (depends on)
Adapter Layer (infrastructure)
```

```java
// ✅ CORRECT - Domain knows nothing about Spring/JPA/Kafka
public class Order { // Pure Java, no framework annotations!
    // ...
}

// ❌ WRONG - Domain polluted with infrastructure concerns
@Entity // JPA annotation in domain!
@Table(name = "orders")
public class Order {
    @Id // JPA in domain!
    private Long id;
}
```

---

# MODULE 3: The Outbox Pattern & Why Domain Has No Spring

## 3.1 The Problem: Dual Write Problem

When you need to save to database AND publish to Kafka, you face a critical issue:

```java
// ❌ WRONG - Dual write without transaction
@Transactional
public void createOrder(CreateOrderCommand command) {
    Order order = new Order(command.getCustomerId(), command.getLines());
    orderRepository.save(order); // 1. Save to DB

    // 2. Publish to Kafka
    kafkaTemplate.send("order-events", order.getDomainEvents());
    // If Kafka is down, DB is saved but event is lost!
    // If JVM crashes after save() but before send(), event is lost!
}
```

**Scenarios where events are LOST:**
1. **Kafka temporarily unavailable** - DB committed, Kafka fails, event lost forever
2. **JVM crash** - Between `save()` and `send()`, event never published
3. **Transaction rollback** - If you wrap both, Kafka isn't transactional with DB
4. **Network partition** - Ack from Kafka lost, application retries = duplicate

## 3.2 The Outbox Pattern Solution

### Concept
Instead of publishing directly to Kafka, write events to an **Outbox table** in the SAME database transaction as your business data. Then have a separate process relay those events to Kafka.

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Use Case   │────▶│   Database   │     │   Kafka     │
│             │     │              │     │             │
│ Save Order  │     │ orders table │     │  Topics     │
│ + Save Event│     │ outbox table │────▶│             │
└─────────────┘     └──────────────┘     └─────────────┘
       │                   ▲                    ▲
       │                   │                    │
       └───────────────────┘                    │
         (Single ACID Transaction)              │
                                               │
                              ┌────────────────┘
                              │
                       ┌──────┴──────┐
                       │   Relay     │
                       │  (Poller)   │
                       └─────────────┘
```

### Why Domain Has No Spring / No Kafka Directly

```java
// ✅ CORRECT - Domain is pure, knows nothing about Spring or Kafka
public class Order {
    private List<DomainEvent> domainEvents = new ArrayList<>();

    public void confirm() {
        this.status = OrderStatus.CONFIRMED;
        // Just register event - don't know HOW it will be published
        registerEvent(new OrderConfirmedEvent(this.id));
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        domainEvents.clear();
    }
}

// ✅ CORRECT - Application service orchestrates, still no Kafka
public class OrderService {
    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository; // Just another repository!

    @Transactional
    public void confirmOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));

        order.confirm(); // Domain logic

        orderRepository.save(order);

        // Save events to outbox - SAME transaction!
        List<OutboxEvent> outboxEvents = order.getDomainEvents().stream()
            .map(event -> new OutboxEvent(
                UUID.randomUUID(),
                event.getClass().getSimpleName(),
                serialize(event),
                Instant.now(),
                false
            ))
            .toList();

        outboxRepository.saveAll(outboxEvents);
        order.clearDomainEvents();
    }
}
```

### Outbox Table Schema

```sql
CREATE TABLE outbox_events (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(255) NOT NULL,      -- e.g., "Order"
    aggregate_id VARCHAR(255) NOT NULL,        -- e.g., order ID
    event_type VARCHAR(255) NOT NULL,          -- e.g., "OrderConfirmedEvent"
    payload JSONB NOT NULL,                    -- Event data as JSON
    metadata JSONB,                            -- Additional metadata
    created_at TIMESTAMP NOT NULL,
    published_at TIMESTAMP,                    -- NULL until published
    retry_count INT DEFAULT 0,
    error_message TEXT,

    INDEX idx_outbox_published (published_at, created_at)
);
```

### Relay Process (Separate Component)

```java
@Component
public class OutboxPoller {

    private final OutboxRepository outboxRepository;
    private final DomainEventPublisher eventPublisher; // Kafka adapter

    @Scheduled(fixedDelay = 5000) // Every 5 seconds
    public void pollAndPublish() {
        List<OutboxEvent> unpublished = outboxRepository
            .findByPublishedAtIsNullOrderByCreatedAtAsc(Pageable.ofSize(100));

        for (OutboxEvent event : unpublished) {
            try {
                DomainEvent domainEvent = deserialize(event);
                eventPublisher.publish(List.of(domainEvent));

                // Mark as published
                event.markAsPublished();
                outboxRepository.save(event);

            } catch (Exception e) {
                event.incrementRetryCount();
                event.setErrorMessage(e.getMessage());
                outboxRepository.save(event);

                if (event.getRetryCount() > 5) {
                    // Move to dead letter queue or alert
                    log.error("Failed to publish event after 5 retries: {}", event.getId());
                }
            }
        }
    }
}
```

### Transaction Log Tailing (Advanced)

Instead of polling, some databases support listening to the transaction log directly:

```java
// Using Debezium for CDC (Change Data Capture)
@Component
public class DebeziumOutboxListener {

    // Debezium reads PostgreSQL WAL (Write-Ahead Log)
    // and publishes changes to Kafka Connect
    // Zero polling delay, minimal overhead
}
```

### Why This Is Better

| Approach | Reliability | Latency | Complexity |
|----------|------------|---------|------------|
| Direct Kafka publish | ❌ Unreliable | ✅ Low | ✅ Simple |
| Outbox + Polling | ✅ Reliable | ⚠️ Medium (5s delay) | ⚠️ Medium |
| Outbox + CDC (Debezium) | ✅ Reliable | ✅ Low | ⚠️ High |
| XA Transactions | ✅ Reliable | ✅ Low | ❌ Very High (don't use) |

---

# MODULE 4: Kafka Deep Dive

## 4.1 Kafka Topic

### Definition
A **Topic** is a logical channel to which producers write and from which consumers read. Topics are partitioned for parallelism and scalability.

```
Topic: "order-events"
┌─────────────────────────────────────────────────────────────┐
│  Partition 0  │  Partition 1  │  Partition 2  │  Partition 3 │
│  ┌─────────┐  │  ┌─────────┐  │  ┌─────────┐  │  ┌─────────┐│
│  │ Offset 0│  │  │ Offset 0│  │  │ Offset 0│  │  │ Offset 0││
│  │ Event 1 │  │  │ Event 4 │  │  │ Event 2 │  │  │ Event 3 ││
│  ├─────────┤  │  ├─────────┤  │  ├─────────┤  │  ├─────────┤│
│  │ Offset 1│  │  │ Offset 1│  │  │ Offset 1│  │  │ Offset 1││
│  │ Event 5 │  │  │ Event 8 │  │  │ Event 6 │  │  │ Event 7 ││
│  ├─────────┤  │  ├─────────┤  │  ├─────────┤  │  ├─────────┤│
│  │ Offset 2│  │  │ Offset 2│  │  │ Offset 2│  │  │ Offset 2││
│  │ Event 9 │  │  │ Event 12│  │  │ Event 10│  │  │ Event 11││
│  └─────────┘  │  └─────────┘  │  └─────────┘  │  └─────────┘│
└─────────────────────────────────────────────────────────────┘
```

### Topic Design Best Practices

```java
// ✅ Event-type topics (preferred for event-driven)
public class KafkaTopics {
    public static final String ORDER_EVENTS = "order-events";      // All order lifecycle
    public static final String PAYMENT_EVENTS = "payment-events";  // All payment events
    public static final String INVENTORY_EVENTS = "inventory-events";
}

// ❌ Don't create per-event topics (too many topics)
public class BadTopics {
    public static final String ORDER_CREATED = "order-created";
    public static final String ORDER_CONFIRMED = "order-confirmed";
    public static final String ORDER_CANCELLED = "order-cancelled";
    // ... explosion of topics
}
```

### Topic Configuration

```java
@Configuration
public class KafkaTopicConfig {

    @Bean
    public NewTopic orderEventsTopic() {
        return TopicBuilder.name("order-events")
            .partitions(6)                    // Parallelism = 6 consumers max
            .replicas(3)                      // Replication factor for HA
            .config(TopicConfig.RETENTION_MS_CONFIG, "604800000") // 7 days
            .config(TopicConfig.CLEANUP_POLICY_CONFIG, "delete")
            .build();
    }

    @Bean
    public NewTopic paymentEventsTopic() {
        return TopicBuilder.name("payment-events")
            .partitions(6)
            .replicas(3)
            .config(TopicConfig.MIN_INSYNC_REPLICAS_CONFIG, "2") // Wait for 2 replicas
            .build();
    }
}
```

## 4.2 Producer

### Configuration

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, DomainEvent> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

        // CRITICAL: Acknowledgment settings
        props.put(ProducerConfig.ACKS_CONFIG, "all"); // Wait for all replicas
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true); // Exactly-once semantics

        // Batching for throughput
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 5);
        props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");

        // Transactions for exactly-once (advanced)
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "order-service-producer-1");

        return new DefaultKafkaProducerFactory<>(props);
    }

    @Bean
    public KafkaTemplate<String, DomainEvent> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### Producer Patterns

```java
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, DomainEvent> kafkaTemplate;

    // Fire and forget (not recommended for critical events)
    public void publishAsync(DomainEvent event) {
        kafkaTemplate.send("order-events", event.getAggregateId(), event);
    }

    // With callback (better)
    public void publishWithCallback(DomainEvent event) {
        kafkaTemplate.send("order-events", event.getAggregateId(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to publish: {}", event, ex);
                    // Handle failure - maybe save to retry queue
                } else {
                    log.info("Published to partition {} offset {}", 
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                }
            });
    }

    // Synchronous (blocks, use sparingly)
    public void publishSync(DomainEvent event) throws Exception {
        SendResult<String, DomainEvent> result = kafkaTemplate
            .send("order-events", event.getAggregateId(), event)
            .get(5, TimeUnit.SECONDS);
    }
}
```

## 4.3 Consumer

### Configuration

```java
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, DomainEvent> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        props.put(JsonDeserializer.TRUSTED_PACKAGES, "com.example.events");

        // Offset management
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // Manual commit!

        // Heartbeat and session
        props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000);
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10000);

        // Max records per poll (tune based on processing time)
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 100);
        props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 300000); // 5 min

        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, DomainEvent> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, DomainEvent> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 3 threads per consumer instance
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        return factory;
    }
}
```

### Consumer Implementation

```java
@Component
@Slf4j
public class OrderEventConsumer {

    private final OrderService orderService;

    @KafkaListener(
        topics = "order-events",
        groupId = "inventory-service",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void onOrderEvent(
            @Payload DomainEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment acknowledgment) {

        log.info("Received event: {} from partition {} offset {}", 
            event.getClass().getSimpleName(), partition, offset);

        try {
            if (event instanceof OrderConfirmedEvent confirmed) {
                orderService.reserveInventory(confirmed.getOrderId());
            } else if (event instanceof OrderCancelledEvent cancelled) {
                orderService.releaseInventory(cancelled.getOrderId());
            }

            acknowledgment.acknowledge(); // Manual commit after success

        } catch (Exception e) {
            log.error("Error processing event: {}", event, e);
            // Don't acknowledge - will retry based on retry policy
            throw e; // Let Spring retry handle it
        }
    }
}
```

## 4.4 Consumer Group

### Concept
A **Consumer Group** is a set of consumers that jointly consume a topic. Kafka distributes partitions among group members.

```
Topic: "order-events" (6 partitions)

Consumer Group: "inventory-service"
┌────────────────────────────────────────────────────────────┐
│  Consumer 1          │  Consumer 2         │  Consumer 3   │
│  (Instance 1)        │  (Instance 2)       │  (Instance 3) │
│                      │                     │               │
│  Partition 0         │  Partition 2        │  Partition 4  │
│  Partition 1         │  Partition 3        │  Partition 5  │
│                      │                     │               │
│  ┌─────────┐         │  ┌─────────┐        │  ┌─────────┐  │
│  │ Events  │         │  │ Events  │        │  │ Events  │  │
│  └─────────┘         │  └─────────┘        │  └─────────┘  │
└────────────────────────────────────────────────────────────┘

If Consumer 2 crashes:
┌────────────────────────────────────────────────────────────┐
│  Consumer 1                    │  Consumer 3               │
│                                │                           │
│  Partition 0                   │  Partition 3              │
│  Partition 1                   │  Partition 4              │
│  Partition 2  ◄── Rebalanced   │  Partition 5              │
│  Partition 3                   │                           │
└────────────────────────────────────────────────────────────┘
```

### Consumer Group Configuration

```java
// Different services = different consumer groups (all get all messages)
@Service
public class InventoryConsumer {
    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void handleForInventory(DomainEvent event) { }
}

@Service
public class NotificationConsumer {
    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleForNotification(DomainEvent event) { }
}

@Service
public class AnalyticsConsumer {
    @KafkaListener(topics = "order-events", groupId = "analytics-service")
    public void handleForAnalytics(DomainEvent event) { }
}

// Same service scaled horizontally = same groupId (share partitions)
@Service
public class PaymentConsumer {
    @KafkaListener(topics = "payment-events", groupId = "payment-service")
    public void handle(DomainEvent event) { }
}
// Deploy 3 instances with groupId "payment-service" → partitions distributed
```

### Partition Assignment Strategy

```java
// Range assignment (default) - assigns consecutive partitions
// RoundRobin - distributes evenly
// Sticky - minimizes movement during rebalance
// CooperativeSticky - incremental rebalancing (Kafka 2.4+)

props.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG, 
    CooperativeStickyAssignor.class.getName());
```

---

# MODULE 5: Payment Flow & Async Boundaries

## 5.1 Payment Flow Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Client    │────▶│ Order Service│────▶│Payment Service│────▶│   Stripe     │
│             │     │              │     │              │     │   Gateway    │
└─────────────┘     └──────┬───────┘     └──────┬───────┘     └──────────────┘
                           │                    │
                           │  OrderConfirmed    │  PaymentProcessed
                           │  Event             │  Event
                           ▼                    ▼
                    ┌──────────────┐     ┌──────────────┐
                    │    Kafka     │     │   Kafka      │
                    │ order-events │     │ payment-events│
                    └──────────────┘     └──────────────┘
```

## 5.2 Async Boundary Definition

An **Async Boundary** is where synchronous processing ends and asynchronous begins. This is where you switch from request-response to event-driven.

```java
@Service
public class OrderApplicationService {

    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;
    private final PaymentServiceClient paymentClient; // Sync call

    @Transactional
    public OrderId createOrder(CreateOrderCommand command) {
        // 1. Create order (synchronous, in transaction)
        Order order = Order.create(command.getCustomerId(), command.getItems());
        orderRepository.save(order);

        // 2. Save event to outbox (same transaction)
        outboxRepository.save(OutboxEvent.from(order.getDomainEvents()));
        order.clearDomainEvents();

        // 3. Return immediately - don't wait for payment!
        return order.getId();
    }

    // Async handler - triggered by event
    @EventListener
    @Async // Run asynchronously
    public void onOrderCreated(OrderCreatedEvent event) {
        // Initiate payment asynchronously
        paymentClient.initiatePayment(event.getOrderId(), event.getTotalAmount());
    }
}

// Payment Service (separate microservice)
@Service
public class PaymentService {

    @KafkaListener(topics = "order-events", groupId = "payment-service")
    @Transactional
    public void handleOrderConfirmed(OrderConfirmedEvent event) {
        // Process payment
        Payment payment = paymentGateway.charge(event.getOrderId(), event.getAmount());

        // Save payment and publish event
        paymentRepository.save(payment);
        outboxRepository.save(OutboxEvent.from(payment.getDomainEvents()));
    }
}
```

## 5.3 Payment Allocation: Penalty → Interest → Principal

This is a common loan/debt payment allocation strategy where incoming payments are applied in a specific priority order.

### Business Rules

```
Payment Received: $1,000
Outstanding Balance:
  - Late Penalties: $150
  - Accrued Interest: $300
  - Principal: $5,000

Allocation Order:
  1. First: Late Penalties ($150) → Remaining: $850
  2. Second: Interest ($300) → Remaining: $550
  3. Third: Principal ($550) → Principal remaining: $4,450
```

### Domain Model

```java
public class LoanAccount { // Aggregate Root
    private LoanId id;
    private Money principalBalance;
    private Money accruedInterest;
    private Money latePenalties;
    private LoanStatus status;
    private List<Payment> paymentHistory;
    private List<DomainEvent> domainEvents;

    // Value Object for allocation result
    public record PaymentAllocation(
        Money penaltyPaid,
        Money interestPaid,
        Money principalPaid,
        Money remainingPayment
    ) {}

    public PaymentAllocation applyPayment(Money paymentAmount) {
        if (status == LoanStatus.CLOSED) {
            throw new IllegalStateException("Loan is already closed");
        }

        Money remaining = paymentAmount;
        Money penaltyPaid = Money.zero(Currency.USD);
        Money interestPaid = Money.zero(Currency.USD);
        Money principalPaid = Money.zero(Currency.USD);

        // 1. Apply to penalties first
        if (latePenalties.isGreaterThan(Money.zero(Currency.USD))) {
            Money penaltyToPay = remaining.isGreaterThan(latePenalties) 
                ? latePenalties 
                : remaining;
            latePenalties = latePenalties.subtract(penaltyToPay);
            penaltyPaid = penaltyToPay;
            remaining = remaining.subtract(penaltyToPay);
        }

        // 2. Apply to interest second
        if (remaining.isGreaterThan(Money.zero(Currency.USD)) && 
            accruedInterest.isGreaterThan(Money.zero(Currency.USD))) {
            Money interestToPay = remaining.isGreaterThan(accruedInterest)
                ? accruedInterest
                : remaining;
            accruedInterest = accruedInterest.subtract(interestToPay);
            interestPaid = interestToPay;
            remaining = remaining.subtract(interestToPay);
        }

        // 3. Apply to principal last
        if (remaining.isGreaterThan(Money.zero(Currency.USD))) {
            principalPaid = remaining;
            principalBalance = principalBalance.subtract(remaining);
            remaining = Money.zero(Currency.USD);
        }

        // Record payment
        Payment payment = new Payment(
            PaymentId.generate(),
            paymentAmount,
            penaltyPaid,
            interestPaid,
            principalPaid,
            Instant.now()
        );
        paymentHistory.add(payment);

        // Check if loan is paid off
        if (principalBalance.equals(Money.zero(Currency.USD)) &&
            accruedInterest.equals(Money.zero(Currency.USD)) &&
            latePenalties.equals(Money.zero(Currency.USD))) {
            status = LoanStatus.CLOSED;
            registerEvent(new LoanPaidOffEvent(this.id));
        }

        registerEvent(new PaymentAppliedEvent(
            this.id, payment.getId(), paymentAmount,
            penaltyPaid, interestPaid, principalPaid
        ));

        return new PaymentAllocation(penaltyPaid, interestPaid, principalPaid, remaining);
    }

    // Business method to accrue interest
    public void accrueInterest(Money interestAmount) {
        if (status != LoanStatus.ACTIVE) {
            throw new IllegalStateException("Can only accrue interest on active loans");
        }
        accruedInterest = accruedInterest.add(interestAmount);
        registerEvent(new InterestAccruedEvent(this.id, interestAmount));
    }

    // Business method to add penalty
    public void addLatePenalty(Money penaltyAmount) {
        if (status != LoanStatus.ACTIVE) {
            throw new IllegalStateException("Can only penalize active loans");
        }
        latePenalties = latePenalties.add(penaltyAmount);
        registerEvent(new LatePenaltyAppliedEvent(this.id, penaltyAmount));
    }
}
```

### Application Service

```java
@Service
public class LoanApplicationService {

    private final LoanRepository loanRepository;
    private final OutboxRepository outboxRepository;
    private final InterestCalculator interestCalculator;

    @Transactional
    public PaymentResult processPayment(LoanId loanId, Money amount) {
        LoanAccount loan = loanRepository.findById(loanId)
            .orElseThrow(() -> new LoanNotFoundException(loanId));

        // Domain logic handles allocation
        PaymentAllocation allocation = loan.applyPayment(amount);

        loanRepository.save(loan);
        outboxRepository.save(OutboxEvent.from(loan.getDomainEvents()));
        loan.clearDomainEvents();

        return PaymentResult.success(allocation);
    }

    // Scheduled job for interest accrual
    @Scheduled(cron = "0 0 0 * * ?") // Daily at midnight
    @Transactional
    public void accrueDailyInterest() {
        List<LoanAccount> activeLoans = loanRepository.findByStatus(LoanStatus.ACTIVE);

        for (LoanAccount loan : activeLoans) {
            Money dailyInterest = interestCalculator.calculateDailyInterest(loan);
            if (dailyInterest.isGreaterThan(Money.zero(Currency.USD))) {
                loan.accrueInterest(dailyInterest);
                loanRepository.save(loan);
                outboxRepository.save(OutboxEvent.from(loan.getDomainEvents()));
                loan.clearDomainEvents();
            }
        }
    }
}
```

### Event Flow for Payment

```
1. Client → POST /loans/{id}/payments {amount: 1000}

2. LoanService → LoanAccount.applyPayment($1000)
   → PaymentAppliedEvent generated
   → If fully paid: LoanPaidOffEvent generated

3. Save to DB + Outbox (same transaction)

4. OutboxPoller reads event

5. Publish to Kafka "loan-events" topic

6. Consumers react:
   - Accounting Service: Update general ledger
   - Notification Service: Send payment confirmation email
   - Credit Service: Update credit score
   - Reporting Service: Update dashboards
```

---

# MODULE 6: Why NOT Publish to Kafka Inside Use Case After save()

## 6.1 The Fundamental Problem

```java
@Service
public class BadOrderService {

    @Autowired private OrderRepository orderRepository;
    @Autowired private KafkaTemplate<String, Event> kafkaTemplate;

    @Transactional
    public void createOrder(CreateOrderCommand cmd) {
        Order order = new Order(cmd);
        orderRepository.save(order); // Transaction committed here

        // DANGER ZONE STARTS
        // JVM could crash here!
        // Network could fail here!
        // Kafka could be down here!

        kafkaTemplate.send("order-events", order.getEvent());
        // DANGER ZONE ENDS
    }
}
```

## 6.2 Failure Scenarios

### Scenario 1: Kafka Unavailable
```
1. DB Transaction commits ✓
2. Kafka connection timeout ✗
3. Exception thrown
4. HTTP 500 returned to client
5. Client retries → Duplicate order created!
6. Original event NEVER published → System inconsistent
```

### Scenario 2: JVM Crash
```
1. DB Transaction commits ✓
2. JVM crashes (OOM, killed by K8s, hardware failure)
3. Event NEVER published
4. Order exists in DB but no one knows about it
5. Inventory not reserved, payment not processed
```

### Scenario 3: Transaction Rollback After Kafka Send
```java
@Transactional
public void createOrder(CreateOrderCommand cmd) {
    Order order = new Order(cmd);
    orderRepository.save(order);

    kafkaTemplate.send("order-events", order.getEvent()); // Sent!

    // Some other operation fails
    inventoryService.reserve(order); // Throws exception!
    // Transaction rolls back → Order NOT saved
    // But event was ALREADY published! → Phantom event!
}
```

## 6.3 The Outbox Solution (Recap)

```java
@Service
public class GoodOrderService {

    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;

    @Transactional
    public void createOrder(CreateOrderCommand cmd) {
        Order order = new Order(cmd);
        orderRepository.save(order);

        // Same transaction - atomic!
        outboxRepository.save(OutboxEvent.from(order.getDomainEvents()));
        order.clearDomainEvents();

        // Transaction commits = both saved or both rolled back
    }
}

// Separate process - eventually consistent
@Component
public class OutboxRelay {
    @Scheduled(fixedDelay = 5000)
    public void poll() {
        List<OutboxEvent> events = outboxRepository.findUnpublished();
        for (OutboxEvent event : events) {
            kafkaTemplate.send(event.getTopic(), event.getPayload());
            event.markPublished();
            outboxRepository.save(event);
        }
    }
}
```

## 6.4 Comparison Matrix

| Scenario | Direct Kafka | Outbox Pattern |
|----------|-------------|----------------|
| Kafka down during send | ❌ Event lost | ✅ Saved in outbox, retried later |
| JVM crash after save | ❌ Event lost | ✅ In outbox, picked up after restart |
| Transaction rollback | ❌ Phantom event | ✅ Rolled back with transaction |
| Exactly-once semantics | ❌ Hard to guarantee | ✅ Achievable with idempotency |
| Latency | ✅ Low | ⚠️ Higher (polling delay) |
| Complexity | ✅ Simple | ⚠️ Requires relay component |
| Testability | ❌ Hard (needs Kafka) | ✅ Easy (just check outbox table) |

---

# MODULE 7: Complete Implementation Example

## 7.1 Project Structure

```
order-service/
├── domain/
│   ├── model/
│   │   ├── Order.java              # Aggregate Root
│   │   ├── OrderLine.java          # Entity
│   │   ├── OrderStatus.java        # Enum
│   │   ├── Money.java              # Value Object
│   │   ├── Quantity.java           # Value Object
│   │   └── events/
│   │       ├── OrderCreatedEvent.java
│   │       ├── OrderConfirmedEvent.java
│   │       └── OrderCancelledEvent.java
│   ├── repository/
│   │   └── OrderRepository.java    # Port
│   └── service/
│       └── PricingService.java     # Domain Service
│
├── application/
│   ├── port/
│   │   ├── inbound/
│   │   │   └── OrderUseCase.java   # Primary Port
│   │   └── outbound/
│   │       ├── OutboxRepository.java
│   │       └── EventPublisher.java
│   ├── service/
│   │   └── OrderApplicationService.java
│   └── dto/
│       ├── CreateOrderCommand.java
│       └── OrderDto.java
│
├── infrastructure/
│   ├── persistence/
│   │   ├── JpaOrderRepository.java       # Adapter
│   │   ├── JpaOutboxRepository.java      # Adapter
│   │   └── entity/
│   │       ├── OrderEntity.java
│   │       └── OutboxEventEntity.java
│   ├── messaging/
│   │   ├── KafkaEventPublisher.java      # Adapter
│   │   └── OutboxPoller.java
│   └── web/
│       └── OrderController.java          # Adapter
│
└── config/
    └── KafkaConfig.java
```

## 7.2 Full Code Example

```java
// ============ DOMAIN LAYER ============

public class Order {
    private OrderId id;
    private CustomerId customerId;
    private List<OrderLine> lines;
    private Money totalAmount;
    private OrderStatus status;
    private List<DomainEvent> domainEvents = new ArrayList<>();

    public static Order create(CustomerId customerId, List<OrderLine> lines) {
        Order order = new Order();
        order.id = OrderId.generate();
        order.customerId = customerId;
        order.lines = new ArrayList<>(lines);
        order.totalAmount = calculateTotal(lines);
        order.status = OrderStatus.PENDING;
        order.registerEvent(new OrderCreatedEvent(order.id, customerId, order.totalAmount));
        return order;
    }

    public void confirm() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Order must be pending");
        }
        status = OrderStatus.CONFIRMED;
        registerEvent(new OrderConfirmedEvent(this.id, this.totalAmount));
    }

    private void registerEvent(DomainEvent event) {
        domainEvents.add(event);
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        domainEvents.clear();
    }
}

// ============ APPLICATION LAYER ============

public interface OrderUseCase {
    OrderId createOrder(CreateOrderCommand command);
    void confirmOrder(OrderId orderId);
}

@Service
@Transactional
public class OrderApplicationService implements OrderUseCase {

    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;

    @Override
    public OrderId createOrder(CreateOrderCommand command) {
        List<OrderLine> lines = command.getItems().stream()
            .map(item -> new OrderLine(
                new ProductId(item.getProductId()),
                new Quantity(item.getQuantity()),
                Money.of(item.getPrice(), Currency.USD)
            ))
            .toList();

        Order order = Order.create(
            new CustomerId(command.getCustomerId()),
            lines
        );

        orderRepository.save(order);
        saveEventsToOutbox(order);

        return order.getId();
    }

    @Override
    public void confirmOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));

        order.confirm();
        orderRepository.save(order);
        saveEventsToOutbox(order);
    }

    private void saveEventsToOutbox(Order order) {
        List<OutboxEvent> events = order.getDomainEvents().stream()
            .map(event -> new OutboxEvent(
                UUID.randomUUID(),
                "Order",
                order.getId().getValue(),
                event.getClass().getSimpleName(),
                serialize(event),
                Instant.now()
            ))
            .toList();

        outboxRepository.saveAll(events);
        order.clearDomainEvents();
    }

    private String serialize(DomainEvent event) {
        try {
            return new ObjectMapper().writeValueAsString(event);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Failed to serialize event", e);
        }
    }
}

// ============ INFRASTRUCTURE LAYER ============

@Entity
@Table(name = "outbox_events")
public class OutboxEventEntity {
    @Id
    private UUID id;
    private String aggregateType;
    private String aggregateId;
    private String eventType;
    @Column(columnDefinition = "jsonb")
    private String payload;
    private Instant createdAt;
    private Instant publishedAt;
    private int retryCount;
}

@Repository
public class JpaOutboxRepository implements OutboxRepository {

    private final SpringDataOutboxRepository jpaRepository;
    private final OutboxMapper mapper;

    @Override
    public void saveAll(List<OutboxEvent> events) {
        List<OutboxEventEntity> entities = events.stream()
            .map(mapper::toEntity)
            .toList();
        jpaRepository.saveAll(entities);
    }

    @Override
    public List<OutboxEvent> findUnpublished(int limit) {
        return jpaRepository.findByPublishedAtIsNullOrderByCreatedAtAsc(Pageable.ofSize(limit))
            .stream()
            .map(mapper::toDomain)
            .toList();
    }

    @Override
    public void markAsPublished(UUID eventId) {
        jpaRepository.findById(eventId).ifPresent(entity -> {
            entity.setPublishedAt(Instant.now());
            jpaRepository.save(entity);
        });
    }
}

@Component
public class KafkaEventPublisher implements EventPublisher {

    private final KafkaTemplate<String, String> kafkaTemplate;
    private final ObjectMapper objectMapper;

    @Override
    public void publish(OutboxEvent event) {
        String topic = resolveTopic(event.getEventType());

        kafkaTemplate.send(topic, event.getAggregateId(), event.getPayload())
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    throw new EventPublishException("Failed to publish event", ex);
                }
            });
    }

    private String resolveTopic(String eventType) {
        return eventType.toLowerCase().replace("event", "") + "-events";
    }
}

@Component
public class OutboxPoller {

    private final OutboxRepository outboxRepository;
    private final EventPublisher eventPublisher;

    @Scheduled(fixedDelay = 5000)
    @Transactional
    public void pollAndPublish() {
        List<OutboxEvent> events = outboxRepository.findUnpublished(100);

        for (OutboxEvent event : events) {
            try {
                eventPublisher.publish(event);
                outboxRepository.markAsPublished(event.getId());
            } catch (Exception e) {
                log.error("Failed to publish event {}", event.getId(), e);
                // Retry logic or dead letter queue
            }
        }
    }
}

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderUseCase orderUseCase;

    @PostMapping
    public ResponseEntity<OrderId> create(@RequestBody CreateOrderRequest request) {
        OrderId id = orderUseCase.createOrder(mapToCommand(request));
        return ResponseEntity.status(HttpStatus.CREATED).body(id);
    }

    @PostMapping("/{orderId}/confirm")
    public ResponseEntity<Void> confirm(@PathVariable String orderId) {
        orderUseCase.confirmOrder(new OrderId(orderId));
        return ResponseEntity.ok().build();
    }
}
```

---

# MODULE 8: Summary & Best Practices

## Key Takeaways

### 1. Aggregate Design
- ✅ Small boundaries, reference others by ID
- ✅ Enforce invariants in constructors and methods
- ✅ Expose domain events, don't publish directly

### 2. Hexagonal Architecture
- ✅ Domain has NO dependencies on frameworks
- ✅ Ports define contracts, adapters implement them
- ✅ Dependency direction: Adapter → Application → Domain

### 3. Outbox Pattern
- ✅ NEVER publish to Kafka inside use case after save()
- ✅ Write events to outbox table in same transaction
- ✅ Use separate relay process to publish to Kafka
- ✅ Domain knows nothing about Spring or Kafka

### 4. Kafka
- ✅ Topics = logical channels, partitions = parallelism
- ✅ Consumer groups = load balancing + failover
- ✅ Manual offset commit after successful processing
- ✅ Idempotent producers for exactly-once

### 5. Payment Allocation
- ✅ Model business rules explicitly (penalty → interest → principal)
- ✅ Use value objects for money and quantities
- ✅ Generate domain events for each allocation
- ✅ Async boundaries at service boundaries

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Solution |
|-------------|-------------|----------|
| Anemic Domain Model | Logic in services, not domain | Rich domain models with behavior |
| Direct Kafka in Use Case | Unreliable, dual write problem | Outbox pattern |
| Spring in Domain | Framework lock-in, untestable | Pure Java domain |
| Huge Aggregates | Performance issues, concurrency | Small, focused aggregates |
| Shared Database | Tight coupling | Database per service |
| Synchronous Chains | Cascading failures | Async events, sagas |

---

*Course Version 1.0 | Advanced DDD & Distributed Systems*
