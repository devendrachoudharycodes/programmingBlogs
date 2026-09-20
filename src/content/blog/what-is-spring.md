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

When starting out with modern Java development, you inevitably run into **Spring**. It is the standard framework for enterprise Java applications. But before diving straight into `@RestController`, `@Autowired`, or `@SpringBootApplication`, it is essential to understand *why* Spring exists, *what problem* it was designed to solve, and *how* to think about the Spring container.

---

## 1. The Historical Problem: Early Java EE Complexity

In the late 1990s and early 2000s, Java Enterprise Edition (J2EE) introduced **Enterprise JavaBeans (EJB 1.x / 2.x)** to handle distributed enterprise applications. While EJB aimed to handle transactions, security, and persistence out of the box, it introduced severe drawbacks:

1. **Heavyweight & Invasive Code**: Components had to extend framework-specific classes and implement complex interface methods, binding business logic directly to J2EE APIs.
2. **Difficult Unit Testing**: Testing EJB components outside a heavy application server (like WebSphere or WebLogic) was nearly impossible.
3. **Tight Coupling**: Objects managed their own dependencies directly (e.g., using `new` operators or manual lookup services like JNDI).

In 2002, **Rod Johnson** published his landmark book, *Expert One-on-One J2EE Design and Development*, where he proposed a lightweight framework centered around **Plain Old Java Objects (POJOs)**. This code became the foundation of the Spring Framework, released as open-source in 2003.

---

## 2. Inversion of Control (IoC) and Dependency Injection (DI)

To understand Spring, you must understand the core principle it relies on: **Inversion of Control (IoC)**.

### Traditional Control Flow
In traditional programming, a class creates and manages its own dependencies using the `new` keyword:

```java
public class OrderService {
    private PaymentProcessor paymentProcessor;

    public OrderService() {
        // Tightly coupled: OrderService controls instantiation
        this.paymentProcessor = new CreditCardPaymentProcessor();
    }
}
```

Here, `OrderService` is tightly coupled to `CreditCardPaymentProcessor`. If you want to switch to `PayPalPaymentProcessor` or use a Mock processor during unit testing, you have to modify `OrderService`.

### Inverted Control Flow (Dependency Injection)
With **Inversion of Control**, the class no longer instantiates its dependencies. Instead, dependencies are "injected" from the outside:

```java
public class OrderService {
    private final PaymentProcessor paymentProcessor;

    // Control inverted: OrderService receives dependency via constructor
    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}
```

Now, `OrderService` only depends on the `PaymentProcessor` interface. The responsibility of creating the dependency and wiring it into `OrderService` is delegated to an external entity: **the Spring Container**.

---

## 3. The Spring Container & Spring Beans

At the heart of the Spring Framework is the **Spring Container** (represented primarily by the `ApplicationContext` interface).

```
   ┌────────────────────────────────────────────────────────┐
   │                    Spring Container                    │
   │                  (ApplicationContext)                  │
   │                                                        │
   │   ┌────────────────────┐      ┌────────────────────┐   │
   │   │  PaymentProcessor  │ ───> │    OrderService    │   │
   │   │       (Bean)       │      │       (Bean)       │   │
   │   └────────────────────┘      └────────────────────┘   │
   └────────────────────────────────────────────────────────┘
```

### What is a Spring Bean?
A **Spring Bean** is simply a Java object that is instantiated, configured, assembled, and managed by the Spring IoC container.

### How the Container Works
1. **Configuration**: You define your beans using Java Configuration (`@Configuration` and `@Bean`), Annotations (`@Component`, `@Service`, `@Repository`), or XML.
2. **Instantiation & Wiring**: Upon application startup, the container reads configuration metadata, instantiates the required beans, and injects dependencies where needed.
3. **Lifecycle Management**: The container manages bean initialization, scope (e.g., Singleton vs. Prototype), and teardown.

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentProcessor paymentProcessor() {
        return new CreditCardPaymentProcessor();
    }

    @Bean
    public OrderService orderService(PaymentProcessor paymentProcessor) {
        return new OrderService(paymentProcessor);
    }
}
```

---

## 4. The Mental Model

When writing Spring applications, adopt this fundamental mental model:

1. **Think in Contracts (Interfaces)**: Design components around interfaces so dependencies can be swapped or mocked easily.
2. **Let Spring Handle Object Creation**: Avoid using `new` for business logic services, repositories, or components. Mark them with Spring annotations or declare them in configuration classes.
3. **Declarative Infrastructure**: Instead of writing boilerplate code for database transaction management, security checks, or REST endpoints, describe *what* you want using annotations (`@Transactional`, `@PreAuthorize`, `@GetMapping`), and let Spring handle *how* it happens.

---

## Summary

Spring turned enterprise Java development on its head by replacing heavy, invasive server dependencies with lightweight POJOs managed by an IoC container. By understanding **Dependency Injection**, the **Spring Container**, and the **Bean Lifecycle**, you have the foundation needed to master the rest of the Spring ecosystem.
