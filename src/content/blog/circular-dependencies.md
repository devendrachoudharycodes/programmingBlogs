---
title: "Circular Dependencies: Causes, Three-Level Cache, and Resolution"
description: "Part 14 of the Spring Core series. Deep dive into circular dependency cycles (A -> B -> A), why constructor cycles fail fast, and how Spring's three-level cache resolves setter cycles."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 14

tags:
  - java
  - spring
  - spring-core
  - circular-dependency
  - three-level-cache

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/circular-dependencies/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 14 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 45 min / 50 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Circular dependency graph, BeanCurrentlyInCreationException, three-level singleton cache, early singleton exposure, constructor vs setter cycle handling |
| **Concepts unlocked** | Part 19 (Complete Bean Lifecycle), Part 24 (AOP Proxies), Part 46 (Singleton Registry Internals) |

## 1. Prerequisites

**Spring concepts required**
- Bean lifecycle, injection styles, autowiring (Parts 2, 8, 10 & 11)

**Previous blogs required**
- Part 8 (*Bean Creation and Initialization*)
- Part 11 (*Constructor vs Setter vs Field Injection*)

---

## 2. What You Will Learn

- What a circular dependency (`Class A` needs `Class B`, `Class B` needs `Class A`) is
- Why constructor circular dependencies fail immediately with `BeanCurrentlyInCreationException`
- Why constructor cycles represent critical architectural smells
- How Spring resolves setter/field circular dependencies using early singleton exposure
- The conceptual mechanism behind Spring's **Three-Level Singleton Cache**
- Architectural techniques to refactor circular dependencies

---

## 3. The Circular Dependency Problem

```text
       ┌──────────────────────┐
       │   OrderService (A)   │
       └──────────┬───────────┘
                  │
        Requires  │  Requires
        Service B │  Service A
                  ▼
       ┌──────────────────────┐
       │  AuditLogService (B) │
       └──────────────────────┘
```

When `OrderService` needs `AuditLogService` in its constructor, and `AuditLogService` needs `OrderService` in its constructor, neither instance can be allocated first. The container gets stuck in an infinite recursion and throws `BeanCurrentlyInCreationException`.

---

## 4. Why Constructor Cycles Fail Fast

Constructor execution requires passing fully instantiated dependency objects into the `newInstance(...)` invocation.
- To build `A`, Spring must first build `B`.
- To build `B`, Spring must first build `A`.
- Neither raw object allocation can begin without the other existing. Therefore, constructor cycles **cannot** be resolved by Spring and fail at startup.

---

## 5. How Setter/Field Cycles Are Resolved (Three-Level Cache)

For setter or field injection, Spring can instantiate raw object `A` via its default no-arg constructor **before** populating `A`'s dependencies.

Spring uses a 3-level internal caching mechanism inside `DefaultListableBeanFactory` *(implementation detail)*:

```text
Level 1: singletonObjects      (Map<String, Object>)      ── Fully initialized & ready singletons
Level 2: earlySingletonObjects (Map<String, Object>)      ── Raw early instances (pre-initialization)
Level 3: singletonFactories    (Map<String, ObjectFactory>)── Factories providing early proxy/raw references
```

### Resolution Flow for Setter Cycle (`A` ──► `B` ──► `A`):
1. Instantiates raw `A` (no-arg constructor).
2. Exposes an `ObjectFactory` for `A` in **Level 3 Cache** (`singletonFactories`).
3. Populates `A`'s properties ──► requires `B`.
4. Instantiates raw `B` (no-arg constructor).
5. Exposes an `ObjectFactory` for `B` in Level 3 Cache.
6. Populates `B`'s properties ──► requires `A`.
7. Looks up `A`: Finds `A`'s factory in **Level 3 Cache**, promotes early reference of `A` to **Level 2 Cache** (`earlySingletonObjects`), and injects it into `B`.
8. `B` finishes initialization and moves to **Level 1 Cache** (`singletonObjects`).
9. `A` receives completed `B`, finishes initialization, and moves to Level 1 Cache.

---

## 6. Demonstration: Constructor Cycle Failure

**File:** `src/main/java/com/example/springcore/blog14/broken/ServiceA.java`

```java
package com.example.springcore.blog14.broken;

import org.springframework.stereotype.Component;

@Component
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

**File:** `src/main/java/com/example/springcore/blog14/broken/ServiceB.java`

```java
package com.example.springcore.blog14.broken;

import org.springframework.stereotype.Component;

@Component
public class ServiceB {
    private final ServiceA serviceA;

    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}
```

**File:** `src/main/java/com/example/springcore/blog14/CircularFailureDemoApp.java`

```java
package com.example.springcore.blog14;

import org.springframework.beans.factory.BeanCurrentlyInCreationException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class CircularFailureDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog14.broken")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 14 - Constructor Circular Dependency Failure ===");

        try {
            new AnnotationConfigApplicationContext(Config.class);
        } catch (Exception e) {
            System.out.println("Caught Expected Exception: " + e.getClass().getName());
            System.out.println("Message: Constructor circular dependency detected!");
        }
    }
}
```

Expected Output:

```text
=== Blog 14 - Constructor Circular Dependency Failure ===
Caught Expected Exception: org.springframework.beans.factory.UnsatisfiedDependencyException
Message: Constructor circular dependency detected!
```

---

## 7. How to Refactor & Fix Circular Dependencies

1. **Redesign Architecture (Best Solution)**: Extract the shared behavior into a 3rd service (`ServiceC`) that both `ServiceA` and `ServiceB` depend on.
2. **Use `@Lazy` on Constructor Parameter**: Tells Spring to inject a lazy proxy instead of the real object immediately.
3. **Switch to Setter Injection**: Allows early singleton exposure via the three-level cache.

---

## 8. Key Takeaways

1. Constructor circular dependencies cannot be resolved and fail fast with `BeanCurrentlyInCreationException`.
2. Setter/field cycles are resolved for singletons using early singleton exposure via Spring's three-level cache.
3. Circular dependencies are architectural smells that signal tight coupling between components.

---

## 9. Next Blog

**Part 15: Bean Scopes**
We explore `singleton`, `prototype`, `request`, `session`, and `application` scopes, including scoped proxies.

Where you are: **Blog 14 of 49** (Phase 2: Dependency Resolution and Wiring - Complete!).
