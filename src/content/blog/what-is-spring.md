---
title: "What Is Spring? The Problem, the Container, and the Mental Model"
description: "Understand why Spring was created, what problem it solves, and how IoC, dependency injection, and the Spring container work together."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 1

tags:
  - java
  - spring
  - spring-core
  - dependency-injection
  - ioc

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/what-is-spring/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 1 of 49 |
| **Difficulty** | 🟢 Beginner |
| **Reading / Coding / Total** | 30 min / 15 min / ~45 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Why Spring exists, coupling and its costs, IoC and DI at a high level, Framework vs Boot vs MVC, modules and ecosystem, the container mental model |
| **Concepts unlocked** | Part 2 (IoC and DI in depth), Part 3 (beans and the container), and the shared vocabulary for every later post |

## 1. Prerequisites

**Java concepts required**
- Classes, interfaces, constructors, `new`, and composition (one object holding a reference to another)
- Packages and imports
- Running a `main` method
- Basic Maven usage (`mvn compile`), no plugin knowledge needed

**Spring concepts required**
No previous Spring knowledge required. This is the entry point of the series.

**Tooling**
- JDK 17 or newer. Spring Framework 7.0 keeps a Java 17 baseline and supports newer JDKs, up to Java 25.
- Maven 3.9 or newer.

---

## 2. What You Will Learn

- What Spring is, and what it is not
- The concrete problems of tightly coupled code: object creation, dependency management, configuration and testing
- What J2EE-era development looked like and why Spring was created as an alternative
- Why Inversion of Control (IoC) and Dependency Injection (DI) became important
- What manual dependency injection solves, and where it stops scaling
- Spring Framework vs Spring Boot vs Spring MVC vs "Spring Core"
- The major Spring modules and the wider ecosystem
- Where the Spring Core container sits in the picture
- When Spring is the right tool, and when it is unnecessary
- A mental model of the container, and the roadmap of this entire series

---

## 3. Why This Topic Matters

Most developers meet Spring backwards. They start with Spring Boot, paste `@SpringBootApplication`, sprinkle `@Autowired`, and everything works—until it doesn't. Then a `NoSuchBeanDefinitionException` or a `BeanCurrentlyInCreationException` appears, and the annotations offer no explanation.

Everything in Spring, from `@Transactional` to Spring Security, sits on one idea: **a container that builds your objects, wires them together, and can wrap or extend them.** If you understand the problem that container solves, you can *predict* its behavior. If you don't, you memorize annotations and hope.

This blog is deliberately about the problem first. The code is small on purpose, because the goal is to make the need for a container feel obvious before we use one.

---

## 4. Real-World Problem

We will use one business scenario for the whole series: placing an order and paying for it. In later blogs this grows into the `spring-core-learning` project with notifications, repositories and events. For now it is three classes:

```text
OrderService
     ↓
PaymentService
     ↓
PaymentGateway (Stripe)
```

### 4.1 What happens without Spring (or any container)

Here is how this is often written first. Every class creates its own collaborators with `new`.

**File:** `src/main/java/com/example/springcore/blog01/plain/TightlyCoupledApp.java`

```java
package com.example.springcore.blog01.plain;

import java.math.BigDecimal;

public class TightlyCoupledApp {

    static class StripePaymentGateway {
        void charge(String orderId, BigDecimal amount) {
            System.out.println("[Stripe] Charging " + amount + " for order " + orderId);
        }
    }

    static class PaymentService {
        // Hard-wired: PaymentService decides WHICH gateway and HOW to build it.
        private final StripePaymentGateway gateway = new StripePaymentGateway();

        void pay(String orderId, BigDecimal amount) {
            if (amount.signum() <= 0) {
                throw new IllegalArgumentException("Amount must be positive: " + amount);
            }
            gateway.charge(orderId, amount);
            System.out.println("[PaymentService] Payment accepted for " + orderId);
        }
    }

    static class OrderService {
        // Hard-wired: OrderService decides HOW to build PaymentService.
        private final PaymentService paymentService = new PaymentService();

        void placeOrder(String orderId, BigDecimal amount) {
            System.out.println("[OrderService] Placing order " + orderId);
            paymentService.pay(orderId, amount);
            System.out.println("[OrderService] Order " + orderId + " confirmed");
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Blog 01 - Tightly coupled (no DI) ===");
        new OrderService().placeOrder("ORD-1001", new BigDecimal("49.99"));
    }
}
```

Expected output:

```text
=== Blog 01 - Tightly coupled (no DI) ===
[OrderService] Placing order ORD-1001
[Stripe] Charging 49.99 for order ORD-1001
[PaymentService] Payment accepted for ORD-1001
[OrderService] Order ORD-1001 confirmed
```

It works, and for a 50-line program that is fine. The trouble shows up as the system grows. Look at what this design cost us:

| Problem | Where it shows up in the code above |
|---|---|
| **Tight coupling** | `PaymentService` names the concrete class `StripePaymentGateway`. Moving to Razorpay means editing `PaymentService`. |
| **Object creation is scattered** | `new` appears inside business classes. There is no single place that says "these are the objects in my application and how they are built". |
| **Hidden dependencies** | `new OrderService()` gives no hint that it needs a payment service, and transitively a Stripe gateway. The dependency graph is buried inside field initializers. |
| **Dependency management** | Every `OrderService` creates its own `PaymentService`, which creates its own gateway. If the gateway wraps an expensive resource (an HTTP client, a connection pool), you now build many of them. |
| **Configuration problems** | Where would the Stripe API key, the timeout, or the dev/prod switch go? Into `System.getProperty` calls scattered through constructors. |
| **Testing problems** | You cannot unit test `OrderService` without a real `PaymentService` calling a real `StripePaymentGateway`. There is no seam to substitute a fake. |

---

### 4.2 The J2EE-era version of the same problem

Spring was not created because `new` is bad. It was created because the *official* enterprise Java approach of the early 2000s made these problems worse, not better.

> **Legacy / historical context.** This describes J2EE / EJB 2.x development circa 2000 to 2004. It explains why Spring was created. It does not describe modern Jakarta EE, which evolved significantly after that (EJB 3.0, CDI and later).

In that model:
- **Components were invasive.** A business object implemented container interfaces (for example `javax.ejb.SessionBean`), plus home and remote interfaces, plus XML deployment descriptors. Your business logic was welded to the platform.
- **Collaborators were found by lookup, not handed to you.** Code pulled dependencies out of a naming service (JNDI) by string name, the Service Locator pattern:

  ```java
  // Legacy / historical, illustrative only.
  Context ctx = new InitialContext();
  Object ref = ctx.lookup("java:comp/env/ejb/PaymentService");
  PaymentServiceHome home =
          (PaymentServiceHome) PortableRemoteObject.narrow(ref, PaymentServiceHome.class);
  PaymentService paymentService = home.create();
  ```

- **Testing needed a container.** Because components depended on the application server, tests often ran *inside* a deployed server (in-container testing), slow and awkward.
- **Configuration was verbose.** Deployment descriptors in XML for every component.

The ideas that became Spring were published in Rod Johnson's 2002 book *Expert One-on-One J2EE Design and Development*, and Spring Framework 1.0 followed in 2004. Its pitch was: write **plain Java objects (POJOs)**, describe how they connect, and let a lightweight container do the assembly, with no application server required to run or test them.

---

### 4.3 First fix: manual dependency injection

Before introducing any framework, let's fix the design itself. Two changes: depend on an **interface** instead of a concrete class, and **receive** collaborators through the constructor instead of creating them.

These domain classes stay unchanged through most of this blog and are reused by the XML and Java Configuration examples later. They contain **no Spring imports at all**.

**File:** `src/main/java/com/example/springcore/blog01/domain/PaymentGateway.java`

```java
package com.example.springcore.blog01.domain;

import java.math.BigDecimal;

public interface PaymentGateway {

    void charge(String orderId, BigDecimal amount);
}
```

**File:** `src/main/java/com/example/springcore/blog01/domain/StripePaymentGateway.java`

```java
package com.example.springcore.blog01.domain;

import java.math.BigDecimal;

public class StripePaymentGateway implements PaymentGateway {

    @Override
    public void charge(String orderId, BigDecimal amount) {
        System.out.println("[Stripe] Charging " + amount + " for order " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/domain/PaymentService.java`

```java
package com.example.springcore.blog01.domain;

import java.math.BigDecimal;

public class PaymentService {

    private final PaymentGateway paymentGateway;

    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void pay(String orderId, BigDecimal amount) {
        if (amount.signum() <= 0) {
            throw new IllegalArgumentException("Amount must be positive: " + amount);
        }
        paymentGateway.charge(orderId, amount);
        System.out.println("[PaymentService] Payment accepted for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/domain/OrderService.java`

```java
package com.example.springcore.blog01.domain;

import java.math.BigDecimal;

public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder(String orderId, BigDecimal amount) {
        System.out.println("[OrderService] Placing order " + orderId);
        paymentService.pay(orderId, amount);
        System.out.println("[OrderService] Order " + orderId + " confirmed");
    }
}
```

Now `main` becomes the single place that knows how the object graph is assembled. This is called the **composition root**.

**File:** `src/main/java/com/example/springcore/blog01/plain/ManualWiringApp.java`

```java
package com.example.springcore.blog01.plain;

import com.example.springcore.blog01.domain.OrderService;
import com.example.springcore.blog01.domain.PaymentGateway;
import com.example.springcore.blog01.domain.PaymentService;
import com.example.springcore.blog01.domain.StripePaymentGateway;

import java.math.BigDecimal;

public class ManualWiringApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 01 - Manual dependency injection (no container) ===");

        // Composition root: the ONE place that knows the concrete classes and their order.
        PaymentGateway paymentGateway = new StripePaymentGateway();
        PaymentService paymentService = new PaymentService(paymentGateway);
        OrderService orderService = new OrderService(paymentService);

        orderService.placeOrder("ORD-1001", new BigDecimal("49.99"));

        // Same OrderService logic, different collaborator. No source change in the domain classes.
        PaymentGateway fakeGateway = (orderId, amount) ->
                System.out.println("[FakeGateway] Pretending to charge " + amount + " for " + orderId);
        new OrderService(new PaymentService(fakeGateway)).placeOrder("ORD-1002", new BigDecimal("10.00"));
    }
}
```

Expected output:

```text
=== Blog 01 - Manual dependency injection (no container) ===
[OrderService] Placing order ORD-1001
[Stripe] Charging 49.99 for order ORD-1001
[PaymentService] Payment accepted for ORD-1001
[OrderService] Order ORD-1001 confirmed
[OrderService] Placing order ORD-1002
[FakeGateway] Pretending to charge 10.00 for ORD-1002
[PaymentService] Payment accepted for ORD-1002
[OrderService] Order ORD-1002 confirmed
```

What changed compared with the tightly coupled version:
- The **constructor is now honest**: `new OrderService(paymentService)` says exactly what `OrderService` needs.
- `PaymentService` depends on the **abstraction** `PaymentGateway`, so Stripe, Razorpay or a test fake are interchangeable.
- Object creation moved to **one place** (`main`).
- Testing works: we swapped the gateway without touching `PaymentService` or `OrderService`.

This is Dependency Injection, and you did it with no framework. That is an important point: **DI is a design technique, not a Spring feature.**

---

### 4.4 What manual DI still leaves unsolved

The composition root above has three lines of wiring. Now imagine 200 classes:
- **Wiring code explodes.** You hand-order 200 constructors, and every new dependency edits the composition root.
- **Creation order is your problem.** You compute the dependency order yourself.
- **Sharing is your problem.** Should there be one `PaymentService` for the whole application, or one per request? You decide by hand for every object.
- **Lifecycle is your problem.** Initialization after construction and cleanup on shutdown must be coded and ordered manually.
- **Environment-specific configuration is your problem.** Which gateway in dev, which in prod, which properties?
- **Cross-cutting behavior is your problem.** Transactions, security checks, caching and metrics need objects to be *wrapped* by other objects. Doing that by hand for every service means writing decorators everywhere.

Something has to take over this work. A component that **reads a description of your objects and their relationships, builds them in the right order, injects dependencies, manages their lifecycle, and can wrap them with extra behavior**—that is an **IoC container**. Spring is, first and foremost, that container.

---

## 5. Core Concept

### 5.1 What is Spring?

**Spring Framework** is a Java application framework whose foundation is an **Inversion of Control (IoC) container**. On top of that container sit modules for aspect-oriented programming, transaction management, data access, web applications, messaging and testing.

Its programming model has three parts:
1. **Plain Java objects.** Your `PaymentService` does not extend a Spring class or implement a Spring interface. Spring is **non-invasive**: domain code stays framework-agnostic wherever possible.
2. **Configuration metadata.** You describe which objects exist and how they relate (in XML, Java, annotations, or a mix).
3. **Container services.** The container builds, wires, manages and can decorate those objects.

### 5.2 IoC and DI in one paragraph

**Inversion of Control** is the principle: *your code stops controlling how its collaborators are created and found; something else does.* **Dependency Injection** is the most common technique to achieve it: collaborators are *pushed into* an object (constructor, setter or field) instead of *pulled* or created by it. IoC is the broader principle, DI is one mechanism.

### 5.3 How Spring evolved

The container has stayed conceptually the same for two decades. What changed is *how you describe your objects*. That is why this series teaches every concept in three configuration styles.

| Year | Version | What changed |
|---|---|---|
| 2002 | n/a | *Expert One-on-One J2EE Design and Development* publishes the ideas that become Spring |
| 2004 | 1.0 | IoC container with XML bean definitions, AOP, JDBC and transaction abstractions |
| 2006 | 2.0 | XML namespaces and custom schemas, more bean scopes |
| 2007 | 2.5 | **Annotation-driven configuration**: `@Autowired`, `@Component`, component scanning |
| 2009 | 3.0 | **Java-based configuration**: `@Configuration`, `@Bean`; SpEL |
| 2011 | 3.1 | Environment abstraction and profiles |
| 2013 | 4.0 | Java 8 support, `@Conditional`, generics as qualifiers |
| 2014 | Boot 1.0 | Spring Boot arrives: auto-configuration and starters |
| 2017 | 5.0 | Reactive stack (WebFlux), Kotlin support |
| 2022 | 6.0 | Jakarta EE namespace (`jakarta.*`), Java 17 baseline, ahead-of-time groundwork |
| 2025 | 7.0 | Jakarta EE 11 baseline, Java 17 baseline with Java 25 supported, JSpecify null-safety, API versioning |

XML, Java and annotation configuration all still work in Spring Framework 7.0. Most new code uses Java and annotations.

---

### 5.4 Spring Framework vs Spring Boot vs Spring MVC vs "Spring Core"

| Term | What it is | Relationship |
|---|---|---|
| **Spring Framework** | The foundation: IoC container plus modules (AOP, transactions, data access, web, messaging, test). | Everything else builds on it. |
| **Spring Core** | In this series: the **IoC container and its supporting infrastructure** (`BeanFactory`, `ApplicationContext`, lifecycle, scopes, `Environment`, resources, events). | The layer this series studies. |
| **Spring MVC** | The Servlet-based **web framework** module (`spring-webmvc`): controllers, request mapping, view resolution. | A module *inside* Spring Framework. |
| **Spring Boot** | An opinionated **setup layer**: auto-configuration, starters, an embedded web server, production endpoints. | Sits *on top of* Spring Framework. |

The most important distinction: **Spring Boot does not replace the container. It creates and configures one for you.**

---

### 5.5 Major Spring Framework modules

| Group | Main artifacts | Purpose |
|---|---|---|
| **Core Container** | `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, `spring-expression` | IoC container, `ApplicationContext`, resources, events, SpEL. |
| **AOP** | `spring-aop`, `spring-aspects` | Proxy-based aspect-oriented programming, the machinery behind declarative services |
| **Data Access** | `spring-jdbc`, `spring-tx`, `spring-orm`, `spring-r2dbc`, `spring-jms` | JDBC templates, transaction management, ORM integration, messaging |
| **Web** | `spring-web`, `spring-webmvc`, `spring-webflux`, `spring-websocket` | Servlet-based and reactive web stacks |
| **Test** | `spring-test` | Test context framework and mocks |

---

### 5.6 Where Spring Core fits

```text
┌───────────────────────────────────────────────────────────────┐
│  Spring Boot · Spring Data · Spring Security · Spring Cloud   │  ← ecosystem
├───────────────────────────────────────────────────────────────┤
│  Spring MVC / WebFlux │ Transactions │ Data Access │ Messaging│  ← framework modules
├───────────────────────────────────────────────────────────────┤
│                  Spring AOP (proxies)                         │
├───────────────────────────────────────────────────────────────┤
│  ████████████  SPRING CORE CONTAINER (this series)  ████████  │
│  BeanFactory · ApplicationContext · BeanDefinitions ·         │
│  Lifecycle · Scopes · Post-processors · Environment · Events  │
└───────────────────────────────────────────────────────────────┘
```

Every layer above the container is *just more beans* configured, wired and wrapped by that container.

---

## 6. Mental Model

Think of Spring as a **factory manager** with a blueprint. You do not build the machine. You hand over a description ("a `PaymentGateway`, a `PaymentService` that needs a gateway, an `OrderService` that needs a payment service"). The manager builds the parts in the right order, plugs them together, hands you the assembled machine, and later shuts it down cleanly.

**Without a container:** every object builds its own collaborators.

```text
main() ──new──► OrderService ──new──► PaymentService ──new──► StripePaymentGateway
        (each class knows the concrete class of the next one down)
```

**With manual DI:** one place builds everything, top-down.

```text
main() ──► new StripePaymentGateway()
       ──► new PaymentService(gateway)
       ──► new OrderService(paymentService)
```

**With the Spring container:** you describe, Spring builds.

```text
 Application code
        │  asks for "orderService"
        ▼
 ApplicationContext ─────────────────────────────┐
        │  is (or delegates to) a                │ also provides: events, resources,
        ▼                                        │ environment, message resolution
 BeanFactory                                     ┘
        │  holds
        ▼
 Bean Definitions   (the blueprint, loaded from XML, @Configuration classes, @Component scanning)
        │  resolved into
        ▼
 Dependency Graph   orderService → paymentService → paymentGateway
        │  instantiated and wired into
        ▼
 Bean Instances     (managed, shared, lifecycle-aware objects)
```

One sentence to keep: **Spring turns "how objects are created and connected" from code you write into metadata you declare.**

---

## 7. How Spring Works Internally

### 7.1 Conceptual behavior

When an `ApplicationContext` starts, the following happens:
1. **Load configuration metadata:** XML, `@Configuration` class, or component scanning.
2. **Turn metadata into bean definitions:** A **bean definition** is a *recipe*: class, constructor arguments, scope, init and destroy callbacks.
3. **Let extension points adjust recipes:** `BeanFactoryPostProcessor` beans edit definitions before objects are built (e.g. resolving `${...}` property placeholders).
4. **Instantiate singleton beans in dependency order:** For each recipe, the container picks a constructor or factory method, resolves dependencies recursively, injects them, runs lifecycle callbacks, and applies `BeanPostProcessor`s.
5. **Publish the context:** The context announces that it is ready by publishing a refresh event.
6. **On close, destroy beans:** Destruction callbacks run in reverse dependency order.

```text
 Metadata ─► BeanDefinitions ─► (BeanFactoryPostProcessors edit) ─► create beans
                                                                        │
                                       constructor ─► inject ─► callbacks ─► BeanPostProcessors
                                                                        │
                                                                   ready to use ─► close ─► destroy
```

Notice the key rule: **definitions come before instances.** Because the container knows the whole graph *as data* before creating anything, it can validate wiring, decide creation order, and wrap objects.

---

### 7.2 Spring is a library first

Spring does not need an application server. In our examples, *your* `main` method creates the container (`new AnnotationConfigApplicationContext(...)`). No server is started. Spring is simply a library on your classpath.

---

## 8. XML Configuration

> **Legacy / historical configuration.** XML was the original way to configure Spring (since 1.0). We cover it because it provides the clearest picture of "configuration as pure metadata, separate from code".

**File:** `src/main/resources/blog01/beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="paymentGateway"
          class="com.example.springcore.blog01.domain.StripePaymentGateway"/>

    <bean id="paymentService"
          class="com.example.springcore.blog01.domain.PaymentService">
        <constructor-arg ref="paymentGateway"/>
    </bean>

    <bean id="orderService"
          class="com.example.springcore.blog01.domain.OrderService">
        <constructor-arg ref="paymentService"/>
    </bean>

</beans>
```

**File:** `src/main/java/com/example/springcore/blog01/xml/XmlConfigApp.java`

```java
package com.example.springcore.blog01.xml;

import com.example.springcore.blog01.domain.OrderService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.math.BigDecimal;
import java.util.Arrays;

public class XmlConfigApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 01 - XML Configuration ===");

        try (ClassPathXmlApplicationContext context =
                     new ClassPathXmlApplicationContext("blog01/beans.xml")) {

            String[] beanNames = Arrays.stream(context.getBeanDefinitionNames())
                    .filter(name -> !name.contains("."))
                    .sorted()
                    .toArray(String[]::new);
            System.out.println("Application beans: " + Arrays.toString(beanNames));

            OrderService orderService = context.getBean("orderService", OrderService.class);
            orderService.placeOrder("ORD-1001", new BigDecimal("49.99"));

            System.out.println("Same instance on repeated lookup: "
                    + (orderService == context.getBean(OrderService.class)));
        }
    }
}
```

- Metadata is **external** to the code.
- Everything is **string-typed** (class names, bean references).
- Domain classes stay **100% Spring-free**.

---

## 9. Java Configuration

Same three beans, same domain classes, but metadata is written in **Java code**.

**File:** `src/main/java/com/example/springcore/blog01/javaconfig/JavaAppConfig.java`

```java
package com.example.springcore.blog01.javaconfig;

import com.example.springcore.blog01.domain.OrderService;
import com.example.springcore.blog01.domain.PaymentGateway;
import com.example.springcore.blog01.domain.PaymentService;
import com.example.springcore.blog01.domain.StripePaymentGateway;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaAppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    @Bean
    public PaymentService paymentService(PaymentGateway paymentGateway) {
        return new PaymentService(paymentGateway);
    }

    @Bean
    public OrderService orderService(PaymentService paymentService) {
        return new OrderService(paymentService);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/javaconfig/JavaConfigApp.java`

```java
package com.example.springcore.blog01.javaconfig;

import com.example.springcore.blog01.domain.OrderService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.math.BigDecimal;
import java.util.Arrays;

public class JavaConfigApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 01 - Java Configuration ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(JavaAppConfig.class)) {

            String[] beanNames = Arrays.stream(context.getBeanDefinitionNames())
                    .filter(name -> !name.contains("."))
                    .sorted()
                    .toArray(String[]::new);
            System.out.println("Application beans: " + Arrays.toString(beanNames));

            OrderService orderService = context.getBean("orderService", OrderService.class);
            orderService.placeOrder("ORD-1001", new BigDecimal("49.99"));

            System.out.println("Same instance on repeated lookup: "
                    + (orderService == context.getBean(OrderService.class)));
        }
    }
}
```

- **Compile-time type-checked** and IDE refactor-friendly.
- Method parameters express dependencies.
- Domain classes still have **no Spring imports**.

---

## 10. Annotation Configuration

Metadata moves **onto the classes themselves** using annotations (`@Component`).

**File:** `src/main/java/com/example/springcore/blog01/annotation/PaymentGateway.java`

```java
package com.example.springcore.blog01.annotation;

import java.math.BigDecimal;

public interface PaymentGateway {
    void charge(String orderId, BigDecimal amount);
}
```

**File:** `src/main/java/com/example/springcore/blog01/annotation/StripePaymentGateway.java`

```java
package com.example.springcore.blog01.annotation;

import org.springframework.stereotype.Component;
import java.math.BigDecimal;

@Component
public class StripePaymentGateway implements PaymentGateway {

    @Override
    public void charge(String orderId, BigDecimal amount) {
        System.out.println("[Stripe] Charging " + amount + " for order " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/annotation/PaymentService.java`

```java
package com.example.springcore.blog01.annotation;

import org.springframework.stereotype.Component;
import java.math.BigDecimal;

@Component
public class PaymentService {

    private final PaymentGateway paymentGateway;

    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public void pay(String orderId, BigDecimal amount) {
        if (amount.signum() <= 0) {
            throw new IllegalArgumentException("Amount must be positive: " + amount);
        }
        paymentGateway.charge(orderId, amount);
        System.out.println("[PaymentService] Payment accepted for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/annotation/OrderService.java`

```java
package com.example.springcore.blog01.annotation;

import org.springframework.stereotype.Component;
import java.math.BigDecimal;

@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder(String orderId, BigDecimal amount) {
        System.out.println("[OrderService] Placing order " + orderId);
        paymentService.pay(orderId, amount);
        System.out.println("[OrderService] Order " + orderId + " confirmed");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog01/annotation/AnnotationAppConfig.java`

```java
package com.example.springcore.blog01.annotation;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackageClasses = AnnotationAppConfig.class)
public class AnnotationAppConfig {
}
```

**File:** `src/main/java/com/example/springcore/blog01/annotation/AnnotationConfigApp.java`

```java
package com.example.springcore.blog01.annotation;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import java.math.BigDecimal;
import java.util.Arrays;

public class AnnotationConfigApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 01 - Annotation Configuration ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(AnnotationAppConfig.class)) {

            String[] beanNames = Arrays.stream(context.getBeanDefinitionNames())
                    .filter(name -> !name.contains("."))
                    .sorted()
                    .toArray(String[]::new);
            System.out.println("Application beans: " + Arrays.toString(beanNames));

            OrderService orderService = context.getBean("orderService", OrderService.class);
            orderService.placeOrder("ORD-1001", new BigDecimal("49.99"));

            System.out.println("Same instance on repeated lookup: "
                    + (orderService == context.getBean(OrderService.class)));
        }
    }
}
```

---

## 11. Compare the Three Approaches

| Aspect | XML Configuration | Java Configuration | Annotation Configuration |
|---|---|---|---|
| **Location** | Separate XML file | Separate Java class (`@Configuration`) | On classes themselves (`@Component`) |
| **Discovery** | Explicit resource path | Explicit config class | Component scanning (`@ComponentScan`) |
| **Wiring** | `<constructor-arg ref="...">` | `@Bean` method parameters | Constructor parameter types (implicit) |
| **Type Safety** | None (Strings) | Full compile-time checking | Compile-time for code, runtime by type |
| **Domain Imports** | No Spring imports | No Spring imports | Contains `@Component` imports |
| **Third-Party Classes** | Supported | Supported | Not supported (cannot edit code) |
| **Modern Usage** | Legacy | Standard for infrastructure / 3rd party | Standard for application classes |

All three end up as `BeanDefinition`s in the container, which is why runtime container behavior is identical across styles.

---

## 12. Complete Working Example

### 12.1 `pom.xml`

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
    <packaging>jar</packaging>
    <name>spring-core-learning</name>

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

## 13. Common Mistakes

### Mistake 1: Creating a managed object with `new` and expecting injection

```java
@Component
public class ReportJob {

    @Autowired
    private OrderService orderService;

    public void run() {
        orderService.placeOrder("ORD-9", new BigDecimal("5.00")); // NullPointerException!
    }
}

// Bad code:
ReportJob job = new ReportJob();
job.run();
```

**Fix:** Obtain `ReportJob` from the container, or inject it into another bean. Never `new` an object that expects container injection.

### Mistake 2: Scanning the wrong package

```java
@Configuration
@ComponentScan("com.example.springcore.blog01.annotation.impl") // Incorrect package!
public class BrokenConfig { }
```

**Fix:** Use `basePackageClasses = AnnotationAppConfig.class` (type-safe) to avoid package rename errors.

### Mistake 3: Creating a new context repeatedly

```java
public void handleRequest() {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(JavaAppConfig.class);
    ctx.getBean(OrderService.class).placeOrder("ORD-1", new BigDecimal("1.00"));
}
```

**Fix:** Create **one** context at application startup and keep it for the application's lifetime.

---

## 14. Interview Questions

### Beginner

1. **What is Spring?**
   Spring Framework is a Java application framework built around an IoC container that creates, wires and manages your objects.
2. **What is Inversion of Control (IoC)?**
   It is the principle that object creation and dependency wiring are delegated to an external container rather than handled inside business classes.
3. **What is Dependency Injection (DI)?**
   A design technique implementing IoC where an object receives its collaborators from the outside (via constructor, setter, or field).

### Intermediate

1. **What is the difference between Spring Framework and Spring Boot?**
   Spring Framework provides the core IoC container and modules. Spring Boot is an opinionated layer on top that automates configuration and setup.
2. **What does manual DI solve, and what does it not solve?**
   Manual DI removes hard-coded `new` calls and enables test mocks. However, it does not handle wiring volume at scale, creation ordering, lifecycle management, or cross-cutting features like transactions.

### Advanced

1. **Why does Spring separate `BeanDefinition` from bean instances?**
   Holding the object graph as metadata (*recipes*) before creating objects allows Spring to validate the graph, determine creation order, resolve placeholders, and apply proxies.
2. **When you call a method on a bean, is the container involved?**
   No. The call is plain Java unless the bean was wrapped in a proxy (e.g., for `@Transactional` or `@PreAuthorize`), in which case the call routes through generated proxy code.

---

## 15. Key Takeaways

1. **Tight coupling** comes from classes creating their collaborators using `new`.
2. **IoC** is the overall principle; **DI** is the primary mechanism to achieve it.
3. **Spring Framework** is the foundation container; **Spring Boot** is an opinionated setup layer.
4. **Definitions come before instances:** Spring loads metadata as `BeanDefinition` recipes before instantiating objects.
5. XML, Java, and Annotation configurations all produce `BeanDefinition`s inside the container—runtime container behavior is identical across styles.

---

## 16. Practice Exercises

### Exercise 1: Basic
Add a `RazorpayPaymentGateway` implementing `PaymentGateway` to the `domain` package (printing `[Razorpay] Charging ...`). 
- Make the **XML** configuration use it by updating the `class` attribute in `beans.xml`.
- Make the **Java Configuration** use it by updating the body of `paymentGateway()`.
- Did any of `OrderService` or `PaymentService` code need to change?

### Exercise 2: Intermediate
Add a `NotificationService` (prints `[Notification] Confirmation sent for <orderId>`) to `domain`, and inject it into `OrderService` so it runs after payment succeeds. Update `ManualWiringApp`, `beans.xml`, `JavaAppConfig`, and the annotation package. Observe how many files were touched in each style.
