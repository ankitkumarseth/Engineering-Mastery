# Creational Design Patterns in Modern Java: Staff & Principal Guide

Creational design patterns abstract the instantiation process. They make a system independent of how its objects are created, composed, and represented. 

In Staff and Principal engineering interviews, you are not evaluated on whether you can regurgitate textbook UML. You are evaluated on:
1. **Thread-Safety & Memory Model Nuances**: (e.g., CPU instruction reordering in double-checked locking, `volatile` semantics).
2. **Immutability & Defensive Programming**: Thread-safe domain construction (Builder vs. Java Records).
3. **Enterprise Framework Mechanics**: How the Spring container handles bean life cycles vs. the classic GoF Singleton.
4. **Modern Java Evolution**: Replacing boilerplate GoF patterns with lambdas, method references, and functional factories.

---

## 1. The Singleton Pattern: Concurrency & Thread-Safety Deep Dive

The Singleton ensures a class has only one instance and provides a global point of access. In enterprise distributed systems, uncontrolled Singletons introduce tight coupling and testing bottlenecks, but thread-safe caching and hardware connection pools require them.

### A. The Broken vs. Correct Double-Checked Locking (DCL)

```java
// ❌ BROKEN: Anti-Pattern without volatile
public class BrokenSingleton {
    private static BrokenSingleton instance; // Missing volatile!

    private BrokenSingleton() {}

    public static BrokenSingleton getInstance() {
        if (instance == null) {                         // 1st Check (Not synchronized)
            synchronized (BrokenSingleton.class) {
                if (instance == null) {                 // 2nd Check
                    instance = new BrokenSingleton();   // HAZARD: Instruction Reordering!
                }
            }
        }
        return instance;
    }
}
```

> [!CAUTION]
> **Why `instance = new BrokenSingleton();` Fails Without `volatile`:**
> At the bytecode level, object allocation is not an atomic operation. The JVM executes three steps:
> 1. `mem = allocate();` // Allocate memory for the object
> 2. `ctor(mem);`         // Call constructor to initialize fields
> 3. `instance = mem;`    // Assign reference to `instance` variable
> 
> Under the **Java Memory Model (JMM)**, the JIT compiler and CPU out-of-order execution units can reorder steps 2 and 3: `1 -> 3 -> 2`.
> If Thread A assigns `instance = mem` before the constructor finishes, Thread B can execute the 1st check (`instance != null`), obtain a reference to a **partially constructed, uninitialized object**, and crash with an NPE or corrupted state!

#### ✅ The Production-Grade Solution: Volatile Double-Checked Locking
```java
public final class ThreadSafeSingleton {
    // volatile guarantees a happens-before relationship and prevents reordering
    private static volatile ThreadSafeSingleton instance;

    private ThreadSafeSingleton() {
        // Guard against reflection attack
        if (instance != null) {
            throw new IllegalStateException("Instance already exists!");
        }
    }

    public static ThreadSafeSingleton getInstance() {
        ThreadSafeSingleton localRef = instance; // Read volatile once into local variable for performance
        if (localRef == null) {
            synchronized (ThreadSafeSingleton.class) {
                localRef = instance;
                if (localRef == null) {
                    instance = localRef = new ThreadSafeSingleton();
                }
            }
        }
        return localRef;
    }
}
```

---

### B. The Bill Pugh Singleton (Initialization-on-Demand Holder)
If you want lazy initialization without the overhead of `synchronized` or `volatile`, use the JVM's class-loading mechanics:

```java
public final class BillPughSingleton {
    private BillPughSingleton() {}

    // Static nested class is NOT loaded into memory until getInstance() is called
    private static class Holder {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return Holder.INSTANCE; // ClassLoader guarantees thread-safe class initialization
    }
}
```

---

### C. The Joshua Bloch Enum Singleton (The Gold Standard)
If thread safety, reflection safety, and serialization safety are all required:

```java
public enum EnumSingleton {
    INSTANCE;

    // Instance variables & business logic
    private final ConnectionPool pool = new ConnectionPool();

    public void executeQuery(String sql) {
        pool.execute(sql);
    }
}
```
* **Why it is invincible**:
  1. The JVM guarantees an Enum constant is instantiated once per ClassLoader.
  2. JVM reflection APIs explicitly forbid `Constructor.newInstance()` on Enums (throws `IllegalArgumentException: Cannot reflectively create enum objects`).
  3. Serialization is handled natively by the JVM without risking duplicate instances via `readResolve()`.

---

## 2. The Builder Pattern: Immutability & Safe Construction

In domain models (e.g., financial transactions, user profiles), objects often have 5–15 attributes, many optional. Telescoping constructors are unreadable; JavaBeans with setters create **mutable, partially initialized objects** that invite concurrency bugs.

### A. Joshua Bloch's Static Inner Builder
```java
public final class PaymentTransaction {
    // 1. All fields are final for thread-safe immutability
    private final String transactionId;
    private final BigDecimal amount;
    private final Currency currency;
    private final String sourceAccount;
    private final String destinationAccount;
    private final String idempotencyKey;

    private PaymentTransaction(Builder builder) {
        this.transactionId = builder.transactionId;
        this.amount = builder.amount;
        this.currency = builder.currency;
        this.sourceAccount = builder.sourceAccount;
        this.destinationAccount = builder.destinationAccount;
        this.idempotencyKey = builder.idempotencyKey;
    }

    public static class Builder {
        // Mandatory fields
        private final String transactionId;
        private final BigDecimal amount;
        private final Currency currency;

        // Optional fields with sensible defaults
        private String sourceAccount;
        private String destinationAccount;
        private String idempotencyKey;

        public Builder(String transactionId, BigDecimal amount, Currency currency) {
            this.transactionId = Objects.requireNonNull(transactionId, "Transaction ID required");
            this.amount = Objects.requireNonNull(amount, "Amount required");
            this.currency = Objects.requireNonNull(currency, "Currency required");
        }

        public Builder sourceAccount(String sourceAccount) {
            this.sourceAccount = sourceAccount;
            return this;
        }

        public Builder destinationAccount(String destinationAccount) {
            this.destinationAccount = destinationAccount;
            return this;
        }

        public Builder idempotencyKey(String idempotencyKey) {
            this.idempotencyKey = idempotencyKey;
            return this;
        }

        public PaymentTransaction build() {
            // Invariant validation before object creation
            if (amount.compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalArgumentException("Amount must be strictly positive");
            }
            return new PaymentTransaction(this);
        }
    }
}
```

### B. Modern Java 16+ Alternative: Records
In modern Java, records provide built-in immutability, `equals()`, `hashCode()`, and `toString()`. When a record has only a few fields, prefer records over heavy Builder boilerplate:

```java
public record PaymentRequest(
    String transactionId,
    BigDecimal amount,
    Currency currency,
    String idempotencyKey
) {
    // Compact constructor for validation
    public PaymentRequest {
        Objects.requireNonNull(transactionId, "ID cannot be null");
        Objects.requireNonNull(currency, "Currency cannot be null");
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
}
```

---

## 3. Factory Method & Abstract Factory Patterns

### A. Factory Method
Defines an interface for creating an object, but lets subclasses decide which class to instantiate.

```java
// Core Product Interface
public interface PaymentProcessor {
    PaymentResult process(BigDecimal amount);
}

public class StripeProcessor implements PaymentProcessor {
    public PaymentResult process(BigDecimal amount) { /* Stripe API call */ return new PaymentResult(true); }
}

public class MastercardProcessor implements PaymentProcessor {
    public PaymentResult process(BigDecimal amount) { /* Switch ISO 8583 message */ return new PaymentResult(true); }
}

// Creator Factory
public abstract class PaymentProcessorFactory {
    public abstract PaymentProcessor createProcessor();

    // Template method using the factory
    public PaymentResult executeTransaction(BigDecimal amount) {
        PaymentProcessor processor = createProcessor();
        return processor.process(amount);
    }
}
```

### B. Modern Java Functional Factory (Lambda Replacement)
Instead of subclassing multiple Creator factories, modern Java uses functional interfaces (`Supplier<T>` or `Function<K, V>`):

```java
public class ModernPaymentFactory {
    private static final Map<PaymentProvider, Supplier<PaymentProcessor>> REGISTRY = new ConcurrentHashMap<>();

    static {
        REGISTRY.put(PaymentProvider.STRIPE, StripeProcessor::new);
        REGISTRY.put(PaymentProvider.MASTERCARD, MastercardProcessor::new);
    }

    public static PaymentProcessor getProcessor(PaymentProvider provider) {
        Supplier<PaymentProcessor> supplier = REGISTRY.get(provider);
        if (supplier == null) {
            throw new IllegalArgumentException("Unsupported provider: " + provider);
        }
        return supplier.get();
    }
}
```

---

## 4. Staff / Principal Interview Strategy: Creational Patterns

### 1. Spring Singleton vs. GoF Singleton
* **Interview Prompt**: *"What is the difference between a Spring `@Service` Singleton and a GoF Singleton?"*
* **The Principal Answer**:
  > *"A classic GoF Singleton enforces uniqueness per **JVM ClassLoader** through a private constructor and a static instance. A Spring Singleton enforces uniqueness per **Spring ApplicationContext** container. In Spring, if you spin up two application contexts in the same JVM, each will have its own instance. Furthermore, Spring beans are standard classes with public constructors, making them trivial to mock and unit test via dependency injection."*

### 2. Immutability in High-Concurrency Systems
* **Interview Prompt**: *"Why prioritize Builder/Records and immutability over JavaBeans in high-scale distributed systems?"*
* **The Principal Answer**:
  > *"Immutable objects are inherently thread-safe without synchronized locks, volatile variables, or defensive copying. Once constructed, their state cannot be mutated by concurrent worker threads. In high-throughput event processing pipelines (like Kafka consumer threads), passing immutable objects eliminates memory visibility races and allows lock-free parallelism."*
