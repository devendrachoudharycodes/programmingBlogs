---
title: "IoC and DI Fundamentals: Control, Inversion, and Dependency Injection Patterns"
description: "Part 2 of the Spring Core series. What control actually means before and after IoC, how Dependency Injection differs from Service Locator and Factory patterns, and how constructor, setter, and field injection work across XML, Java, and Annotation configuration styles."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 2

tags:
  - java
  - spring
  - spring-core
  - ioc
  - dependency-injection
  - design-patterns

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/ioc-and-di-fundamentals/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 2 of 49 |
| **Difficulty** | 🟢 Beginner |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Control before and after IoC, IoC vs DI, DI vs Service Locator vs Factory, Constructor vs Setter vs Field injection, three wiring styles for injection |
| **Concepts unlocked** | Part 3 (Beans and the Spring IoC Container), Part 10 (Autowiring), Part 11 (Injection style deep dive) |

## 1. Prerequisites

**Java concepts required**
- Classes, interfaces, constructors, methods, and member fields
- Composition (an object holding a reference to another object)
- basic Java access modifiers (`private`, `public`)

**Spring concepts required**
- Understanding why Spring exists as an IoC container (covered in Part 1)

**Previous blogs required**
- Part 1 (*What Is Spring? The Problem, the Container, and the Mental Model*)

**Tooling**
- JDK 17 or newer
- Maven 3.9 or newer

---

## 2. What You Will Learn

- What "control" specifically means in traditional software execution
- What changes when control is inverted (Inversion of Control)
- Why Inversion of Control is a general design principle and Dependency Injection is one mechanism
- How Dependency Injection compares to the **Service Locator** and **Factory** patterns
- The mechanics of **Constructor Injection**, **Setter Injection**, and **Field Injection**
- Why field injection is strongly discouraged in production code
- How constructor and setter injection are configured across **XML**, **Java Configuration**, and **Annotations**
- Common mistakes when adopting DI and how to fix them

---

## 3. Why This Topic Matters

Developers frequently use the terms **Inversion of Control (IoC)** and **Dependency Injection (DI)** interchangeably as if they were synonyms. This leads to confusion when discussing architectural patterns, software testing, or container design.

Understanding the precise boundary between IoC and DI clarifies why Spring is structured the way it is. Furthermore, mastering the three injection mechanisms (constructor, setter, field) prevents subtle runtime errors, immutability bugs, and un-testable code.

---

## 4. Real-World Problem

We continue with our payment processing system: `OrderService` needs `PaymentService`, which needs `PaymentGateway`.

### 4.1 Object Creation and Control Before IoC

In traditional Java execution, a class controls two distinct responsibilities:
1. **Executing its own business logic.**
2. **Managing the lifecycle and lookup of its collaborators.**

**File:** `src/main/java/com/example/springcore/blog02/plain/DirectControlApp.java`

```java
package com.example.springcore.blog02.plain;

import java.math.BigDecimal;

public class DirectControlApp {

    interface PaymentGateway {
        void charge(String orderId, BigDecimal amount);
    }

    static class StripePaymentGateway implements PaymentGateway {
        @Override
        public void charge(String orderId, BigDecimal amount) {
            System.out.println("[Stripe] Charged " + amount + " for " + orderId);
        }
    }

    static class PaymentService {
        // Control 1: PaymentService explicitly controls WHICH class to instantiate
        private final PaymentGateway gateway = new StripePaymentGateway();

        public void pay(String orderId, BigDecimal amount) {
            gateway.charge(orderId, amount);
        }
    }

    static class OrderService {
        // Control 2: OrderService explicitly controls WHEN and HOW to instantiate PaymentService
        private final PaymentService paymentService = new PaymentService();

        public void placeOrder(String orderId, BigDecimal amount) {
            paymentService.pay(orderId, amount);
        }
    }

    public static void main(String[] args) {
        OrderService service = new OrderService();
        service.placeOrder("ORD-2001", new BigDecimal("99.00"));
    }
}
```

In this code, `OrderService` **controls** the creation of `PaymentService`, which **controls** the creation of `StripePaymentGateway`. The execution thread flows downward, and object creation flows downward alongside it.

---

### 4.2 Revisiting the JNDI Service Locator (The Classic EJB Approach)

Before Spring, enterprise Java attempted to solve hard-coded `new` expressions by using a **Service Locator** (such as JNDI).

```java
// Service Locator pattern (legacy enterprise approach)
public class PaymentService {
    private final PaymentGateway gateway;

    public PaymentService() {
        try {
            // Control is still inside PaymentService: it ACTIVELY LOOKS UP its dependency
            Context ctx = new InitialContext();
            this.gateway = (PaymentGateway) ctx.lookup("java:comp/env/service/PaymentGateway");
        } catch (NamingException e) {
            throw new RuntimeException("Could not locate gateway", e);
        }
    }
}
```

**Why Service Locator is not Dependency Injection:**
- In Service Locator, `PaymentService` is still in control of *fetching* its collaborator.
- The dependency is hidden inside the constructor—looking at `new PaymentService()` gives no compile-time clue that a JNDI registry must be running.
- Unit testing requires spinning up a mock JNDI server context.

---

### 4.3 The Factory Pattern Approach

Another common pattern is the **Factory Pattern**:

```java
public class PaymentGatewayFactory {
    public static PaymentGateway getGateway() {
        return new StripePaymentGateway();
    }
}

public class PaymentService {
    private final PaymentGateway gateway;

    public PaymentService() {
        // PaymentService calls a Factory to retrieve its dependency
        this.gateway = PaymentGatewayFactory.getGateway();
    }
}
```

While the Factory pattern decouples `PaymentService` from the concrete `StripePaymentGateway` class, `PaymentService` is still actively pulling the dependency from `PaymentGatewayFactory`. It is still in control of lookup timing.

---

### 4.4 Inverted Control: What Changes

When we invert control (IoC), `PaymentService` stops creating or fetching its dependencies. Instead, dependencies are **provided to it**:

```java
public class PaymentService {
    private final PaymentGateway gateway;

    // Control Inverted: PaymentService passively receives gateway from outside
    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Now, `PaymentService` does not know *where* `gateway` came from, *how* it was instantiated, or *when* it was created. Control over dependency management has been completely inverted.

---

## 5. Core Concept

### 5.1 IoC vs. DI: The Distinction

- **Inversion of Control (IoC)** is a broad software design **principle** where the flow of control is inverted compared to traditional procedural programming. Frameworks (like GUI toolkits, servlet containers, and Spring) embody IoC: the framework calls your application code rather than your code calling the framework (the *Hollywood Principle*: "Don't call us, we'll call you").
- **Dependency Injection (DI)** is a specific **design pattern** and mechanism used to implement IoC for object dependency management. Collaborators are pushed into a dependent object at creation or initialization time.

```text
               ┌─────────────────────────────────────────┐
               │    Inversion of Control (Principle)     │
               └────────────────────┬────────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
┌─────────────────┐       ┌───────────────────┐       ┌─────────────────┐
│   Dependency    │       │   Event-Driven    │       │ Template Method │
│ Injection (DI)  │       │   Frameworks      │       │     Pattern     │
└─────────────────┘       └───────────────────┘       └─────────────────┘
```

---

### 5.2 The Three Types of Dependency Injection

Spring supports three primary mechanisms for injecting dependencies into a bean:

#### 1. Constructor Injection (Recommended)
Dependencies are provided as parameters to the class constructor.

```java
public class PaymentService {
    private final PaymentGateway gateway;

    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

**Advantages:**
- Guarantees **immutability** (fields can be `final`).
- Ensures **valid state**: an instance cannot be created without required dependencies.
- **Unit test friendly**: easy to instantiate in plain JUnit tests using `new PaymentService(mockGateway)`.
- Prevents hidden dependencies.

#### 2. Setter Injection
Dependencies are provided through JavaBean-style setter methods.

```java
public class PaymentService {
    private PaymentGateway gateway;

    public void setPaymentGateway(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

**Advantages:**
- Supports **optional** or **re-configurable** dependencies.
- Allows re-injection or changing dependencies at runtime (rare).

**Disadvantages:**
- Fields cannot be `final`.
- Objects can exist in an incomplete, partially initialized state before setters are invoked.

#### 3. Field Injection (Discouraged)
Dependencies are injected directly into private fields using annotations (e.g., `@Autowired`).

```java
public class PaymentService {
    @Autowired
    private PaymentGateway gateway; // Injected via reflection
}
```

**Why Field Injection is Discouraged in Production:**
1. **Impossible to unit test without Spring or Reflection**: You cannot write `new PaymentService()` in a unit test because `gateway` will remain `null`.
2. **Violates Encapsulation**: Relies on reflection to mutate private fields.
3. **Hides Dependency Count**: Constructor injection becomes visibly messy when a class has 10 dependencies (signaling a violation of Single Responsibility Principle). Field injection hides this smell.
4. **Immutability Impossible**: Fields cannot be `final`.

---

### 5.3 Architectural Patterns Comparison

| Aspect | Manual Object Construction | Factory Pattern | Service Locator Pattern | Dependency Injection (DI) |
|---|---|---|---|---|
| **Control Owner** | Dependent Class | Dependent Class | Dependent Class | External Container / Composition Root |
| **Dependency Fetching** | `new ConcreteClass()` | `Factory.get()` | `Locator.lookup("name")` | Pushed into Constructor/Setter |
| **Testability** | Impossible without code changes | Requires mock factory setup | Requires mock locator setup | Trivial (pass mocks directly) |
| **Compile-time Safety** | Yes | Yes | No (string-based lookups) | Yes (Constructor/Setter parameters) |
| **Coupling** | Tightly coupled to concrete class | Coupled to Factory class | Coupled to Locator registry | Decoupled (depends on interface) |

---

## 6. Mental Model

Think of IoC and DI like an assembly line in a car factory:

```text
TRADITIONAL (No IoC):
[ Car Body ] ───> Executes: "I will build my own engine using new Engine()"
                  (Car Body controls engine creation)

SERVICE LOCATOR:
[ Car Body ] ───> Executes: "I will search the warehouse for 'V8-Engine'"
                  (Car Body controls lookup)

DEPENDENCY INJECTION (IoC Container):
[ Assembly Line Container ]
       ├── Creates [ Engine ]
       └── Injects [ Engine ] ───> [ Car Body(Engine) ]
                  (Car Body receives engine passively)
```

---

## 7. How Spring Works Internally

### 7.1 How Spring Resolves and Injects Dependencies

When Spring encounters a request for a bean (e.g. `OrderService`), the container performs the following sequence:

1. **Inspects Metadata**: Checks XML `<constructor-arg>`, `@Bean` method parameters, or `@Autowired` constructors.
2. **Identifies Required Types**: Determines that `OrderService` requires `PaymentService`.
3. **Recursive Resolution**: Checks if `PaymentService` is already instantiated in the singleton registry.
   - If not, resolves `PaymentService`'s dependencies (`PaymentGateway`) first.
4. **Instantiation**: Invokes the selected constructor or factory method passing resolved dependency instances.
5. **Property Population**: Invokes setters or injects annotated fields for any remaining dependencies.

```text
BeanDefinition Metadata ──► Dependency Graph Resolution ──► Recursive Instantiation ──► Population
```

---

### 7.2 Public Contract vs. Implementation Detail

| Feature | Public Contract (Stable API) | Implementation Detail (Internal) |
|---|---|---|
| Injection Mechanisms | Constructor, Setter, and Field Injection support | `AutowiredAnnotationBeanPostProcessor` class processing `@Autowired` |
| Dependency Resolution | Resolution by type, qualifier, and bean name | `DefaultListableBeanFactory.doResolveDependency()` algorithm |
| Immutability | `final` fields supported via constructor injection | Reflection-based field override (`Field.setAccessible(true)`) |

---

## 8. XML Configuration

We demonstrate **Constructor Injection** and **Setter Injection** in XML.

**File:** `src/main/resources/blog02/beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Concrete Gateway Bean -->
    <bean id="paymentGateway"
          class="com.example.springcore.blog02.domain.StripePaymentGateway"/>

    <!-- Setter Injection Example: Uses <property name="..." ref="..."/> -->
    <bean id="paymentService"
          class="com.example.springcore.blog02.domain.PaymentService">
        <property name="paymentGateway" ref="paymentGateway"/>
    </bean>

    <!-- Constructor Injection Example: Uses <constructor-arg ref="..."/> -->
    <bean id="orderService"
          class="com.example.springcore.blog02.domain.OrderService">
        <constructor-arg ref="paymentService"/>
    </bean>

</beans>
```

- `<constructor-arg ref="...">` invokes the matching constructor parameter.
- `<property name="paymentGateway" ref="...">` invokes `setPaymentGateway(PaymentGateway gateway)`.

---

## 9. Java Configuration

In Java Configuration, constructor and setter injection are expressed directly in `@Bean` methods.

**File:** `src/main/java/com/example/springcore/blog02/javaconfig/JavaAppConfig.java`

```java
package com.example.springcore.blog02.javaconfig;

import com.example.springcore.blog02.domain.OrderService;
import com.example.springcore.blog02.domain.PaymentGateway;
import com.example.springcore.blog02.domain.PaymentService;
import com.example.springcore.blog02.domain.StripePaymentGateway;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaAppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    // Setter Injection in Java Config
    @Bean
    public PaymentService paymentService(PaymentGateway paymentGateway) {
        PaymentService service = new PaymentService();
        service.setPaymentGateway(paymentGateway);
        return service;
    }

    // Constructor Injection in Java Config
    @Bean
    public OrderService orderService(PaymentService paymentService) {
        return new OrderService(paymentService);
    }
}
```

---

## 10. Annotation Configuration

Using annotations, Spring automatically detects constructors or setters.

**File:** `src/main/java/com/example/springcore/blog02/annotation/PaymentService.java`

```java
package com.example.springcore.blog02.annotation;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.math.BigDecimal;

@Component
public class PaymentService {

    private PaymentGateway paymentGateway;

    public PaymentService() {
    }

    // Setter Injection using @Autowired
    @Autowired
    public void setPaymentGateway(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void pay(String orderId, BigDecimal amount) {
        paymentGateway.charge(orderId, amount);
        System.out.println("[PaymentService] Payment executed for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog02/annotation/OrderService.java`

```java
package com.example.springcore.blog02.annotation;

import org.springframework.stereotype.Component;

import java.math.BigDecimal;

@Component
public class OrderService {

    private final PaymentService paymentService;

    // Single constructor: @Autowired is implicit in Spring 4.3+
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder(String orderId, BigDecimal amount) {
        System.out.println("[OrderService] Placing order " + orderId);
        paymentService.pay(orderId, amount);
    }
}
```

---

## 11. Compare the Three Approaches

| Aspect | XML Configuration | Java Configuration | Annotation Configuration |
|---|---|---|---|
| **Constructor Injection** | `<constructor-arg ref="...">` | Pass parameter to `@Bean` method & constructor | Constructor parameter with `@Component` |
| **Setter Injection** | `<property name="..." ref="...">` | Call setter inside `@Bean` method | Put `@Autowired` on setter method |
| **Field Injection** | N/A (Not supported) | N/A (Not supported) | Put `@Autowired` directly on private field |
| **Type Safety** | None (XML strings) | High (Compile-time Java) | High (Compile-time Java) |
| **Immutability (`final`)** | Supported via constructor | Supported via constructor | Supported via constructor |

---

## 12. Complete Working Example

### 12.1 Project Structure

```text
spring-core-learning/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/example/springcore/blog02/
        │       ├── plain/
        │       │   └── DirectControlApp.java
        │       ├── domain/
        │       │   ├── PaymentGateway.java
        │       │   ├── StripePaymentGateway.java
        │       │   ├── PaymentService.java
        │       │   └── OrderService.java
        │       ├── xml/
        │       │   └── XmlConfigApp.java
        │       ├── javaconfig/
        │       │   ├── JavaAppConfig.java
        │       │   └── JavaConfigApp.java
        │       └── annotation/
        │           ├── PaymentGateway.java
        │           ├── StripePaymentGateway.java
        │           ├── PaymentService.java
        │           ├── OrderService.java
        │           ├── AnnotationAppConfig.java
        │           └── AnnotationConfigApp.java
        └── resources/
            └── blog02/
                └── beans.xml
```

### 12.2 `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-core-learning</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.version>7.0.8</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 13. Execution Flow

1. `AnnotationConfigApplicationContext` initializes.
2. Spring scans `com.example.springcore.blog02.annotation`.
3. Finds `StripePaymentGateway` (`@Component`) and registers `stripePaymentGateway` bean definition.
4. Finds `PaymentService` (`@Component`). Sees default constructor + `@Autowired` setter for `PaymentGateway`.
5. Finds `OrderService` (`@Component`). Sees single constructor accepting `PaymentService`.
6. Instantiates `stripePaymentGateway`.
7. Instantiates `PaymentService` and invokes `setPaymentGateway(stripePaymentGateway)`.
8. Instantiates `OrderService(paymentService)`.
9. `OrderService.placeOrder()` executes cleanly.

---

## 14. Common Mistakes

### Mistake 1: NullPointerException when manually instantiating `@Autowired` classes

```java
// BROKEN: Manual instantiation bypasses Spring container!
OrderService service = new OrderService();
service.placeOrder("ORD-1001", new BigDecimal("50.00")); // NullPointerException on paymentService
```

**Fix:** Always retrieve beans from `ApplicationContext` or inject them into other managed beans.

---

### Mistake 2: Using Field Injection and Hiding Dependencies

```java
@Component
public class OrderService {
    @Autowired private PaymentService paymentService;
    @Autowired private InventoryService inventoryService;
    @Autowired private NotificationService notificationService;
    @Autowired private AuditLogger auditLogger;
    // 10+ hidden field injections...
}
```

**Fix:** Switch to Constructor Injection. Large constructor parameter lists immediately alert you that the class has too many responsibilities (SRP violation).

---

### Mistake 3: Passing `ApplicationContext` to Business Classes (Service Locator Anti-Pattern)

```java
@Component
public class OrderService {
    @Autowired
    private ApplicationContext context;

    public void placeOrder(String id, BigDecimal amount) {
        // ANTI-PATTERN: Re-introducing Service Locator inside Spring!
        PaymentService ps = context.getBean(PaymentService.class);
        ps.pay(id, amount);
    }
}
```

**Fix:** Inject `PaymentService` directly into `OrderService` via constructor injection.

---

## 15. Interview Questions

### Beginner

1. **What is the difference between IoC and DI?**
   IoC is a general design principle where framework controls execution flow. DI is a specific pattern implementing IoC by pushing dependencies into objects.
2. **What are the three main types of Dependency Injection?**
   Constructor Injection, Setter Injection, and Field Injection.
3. **Which injection type is recommended by the Spring team and why?**
   Constructor Injection. It enables immutability (`final` fields), guarantees valid initial state, and simplifies unit testing without Spring.

### Intermediate

1. **How does Dependency Injection differ from the Service Locator pattern?**
   In Service Locator, the object actively asks a locator/registry for dependencies (`locator.get(...)`). In DI, dependencies are passively passed into the object.
2. **Is `@Autowired` required on constructors in modern Spring?**
   No. Starting with Spring 4.3, if a class has exactly one constructor, Spring automatically uses it for dependency injection without needing `@Autowired`.

### Advanced

1. **Why is Field Injection considered an anti-pattern?**
   It prevents immutability, hides dependency count smells, violates encapsulation via reflection, and makes plain unit testing impossible without starting a Spring test context or using reflection hacks.
2. **How does Spring handle optional dependencies with Setter Injection vs Constructor Injection?**
   Setter injection naturally supports optional dependencies because setters don't have to be called. With constructor injection, optional dependencies are represented using `Optional<T>`, `@Nullable`, or `@Autowired(required = false)`.

---

## 16. Production Considerations

- **Default to Constructor Injection**: Make dependency fields `private final`.
- **Avoid Field Injection**: Use static analysis tools (like SonarQube or SpotBugs) to flag `@Autowired` on private fields.
- **Keep Domain Logic Container-Agnostic**: Keep domain models free of `org.springframework` imports whenever possible.

---

## 17. When NOT To Use It

- **Simple Data Transfer Objects (DTOs), Entities, and Value Objects**: Objects that hold state per request/database row (e.g. `Order`, `User`) should be instantiated with `new` or builders, not managed by IoC container.
- **Utility Classes with Pure Static Functions**: Math functions or pure string utilities don't need dependency injection or container management.

---

## 18. Key Takeaways

1. **IoC is the Principle, DI is the Pattern**: Control moves from class internals to an external container.
2. **Constructor Injection is King**: Provides immutability, safety, and unit testability.
3. **Avoid Service Locator**: Don't inject `ApplicationContext` to call `getBean()` inside business services.
4. **All 3 Config Styles Produce the Same Container Graph**: XML, Java Config, and Annotations all compile down to `BeanDefinition`s.

---

## 19. Next Blog

**Part 3: Beans and the Spring IoC Container**
Now that we understand IoC and DI, we look deeper at what a **Spring Bean** actually is, how `BeanFactory` and `ApplicationContext` manage bean metadata (`BeanDefinition`), and the 10-step resolution process.

Where you are: **Blog 2 of 49** (Phase 1: IoC and Container Fundamentals).

---

## Practice Exercises

### Exercise 1: Basic
Convert a class using field injection (`@Autowired private PaymentGateway gateway;`) to use **Constructor Injection** with `final` fields. Write a plain JUnit-style test without Spring that instantiates the class using `new`.

### Exercise 2: Intermediate
Create a class `OrderService` that accepts `PaymentService` via Constructor Injection and an optional `NotificationService` via Setter Injection. Verify how Spring injects both when `NotificationService` is present versus absent.

### Exercise 3: Advanced
Build a simple custom mini-container in 50 lines of Java using Reflection (`Class.getDeclaredConstructors()`) that performs constructor dependency injection for two classes without using Spring.

<details>
<summary>Check your answer for Exercise 3</summary>

```java
public class MiniContainer {
    private final Map<Class<?>, Object> singletons = new HashMap<>();

    public <T> void register(Class<T> type) throws Exception {
        Constructor<?> constructor = type.getDeclaredConstructors()[0];
        Class<?>[] paramTypes = constructor.getParameterTypes();
        Object[] params = new Object[paramTypes.length];
        for (int i = 0; i < paramTypes.length; i++) {
            params[i] = singletons.get(paramTypes[i]);
        }
        Object instance = constructor.newInstance(params);
        singletons.put(type, instance);
    }

    public <T> T getBean(Class<T> type) {
        return type.cast(singletons.get(type));
    }
}
```
This shows how Spring inspects constructor parameter types and recursively resolves dependencies before instantiating objects.

</details>
