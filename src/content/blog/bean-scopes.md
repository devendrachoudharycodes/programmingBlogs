---
title: "Bean Scopes: Singleton, Prototype, Web Scopes, and Scoped Proxies"
description: "Part 15 of the Spring Core series. Deep dive into singleton vs prototype bean scopes, prototype lifecycle differences, web scopes, and scoped proxies."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 15

tags:
  - java
  - spring
  - spring-core
  - bean-scopes
  - singleton
  - prototype

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/bean-scopes/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 15 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 50 min / ~90 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Singleton vs prototype scope, web scopes (request, session, application), prototype lifecycle caveats, scoped proxies |
| **Concepts unlocked** | Part 16 (Method Injection and @Lookup), Part 18 (Custom Bean Scopes) |

## 1. Prerequisites

**Spring concepts required**
- Beans, `BeanDefinition`, and container lifecycle (Parts 3, 8 & 14)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 8 (*Bean Creation and Initialization*)

---

## 2. What You Will Learn

- The 6 standard Spring bean scopes and their target environments
- Singleton scope semantics (one shared instance per `ApplicationContext`, not GoF singleton)
- Prototype scope lifecycle caveats: **Why Spring does NOT call destruction callbacks for prototype beans!**
- Web-aware scopes (`request`, `session`, `application`, `websocket`)
- How **Scoped Proxies** allow injecting short-lived beans (e.g. `request`) into long-lived beans (e.g. `singleton`)

---

## 3. Spring Bean Scopes Comparison

| Scope | Applicable Context | Description | Lifetime / Destruction Callbacks |
|---|---|---|---|
| **`singleton` (Default)** | Any Spring Context | One shared instance per `ApplicationContext` container. | Managed by container; `@PreDestroy` runs on context close. |
| **`prototype`** | Any Spring Context | New instance created **every time** bean is requested or injected. | Container instantiates & wires, then **abandons**. No destruction callbacks! |
| **`request`** | Web Context | One instance per HTTP request lifecycle. | Destroyed when HTTP request finishes. |
| **`session`** | Web Context | One instance per HTTP Session. | Destroyed when HTTP session expires/invalidates. |
| **`application`** | Web Context | One instance per `ServletContext`. | Destroyed when web application shuts down. |

---

## 4. Prototype Lifecycle Caveat (CRITICAL)

```text
SINGLETON BEAN:
Creation ──► Injection ──► Init Callbacks ──► Usage ──► Container Close ──► @PreDestroy Runs

PROTOTYPE BEAN:
Creation ──► Injection ──► Init Callbacks ──► Handed to Code ──► Container Forgets ──► @PreDestroy NEVER Runs!
```

Because Spring does not track prototype instances after creation, client code is responsible for cleaning up resources created by prototype beans!

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog15/domain/SingletonCounter.java`

```java
package com.example.springcore.blog15.domain;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("singleton")
public class SingletonCounter {
    private int count = 0;
    public int increment() { return ++count; }
}
```

**File:** `src/main/java/com/example/springcore/blog15/domain/PrototypeTask.java`

```java
package com.example.springcore.blog15.domain;

import jakarta.annotation.PreDestroy;
import org.springframework.config.annotation.Scope;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype")
public class PrototypeTask {

    public PrototypeTask() {
        System.out.println("[PrototypeTask] Instance constructed!");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("[PrototypeTask] Destroy callback executed.");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog15/ScopeDemoApp.java`

```java
package com.example.springcore.blog15;

import com.example.springcore.blog15.domain.PrototypeTask;
import com.example.springcore.blog15.domain.SingletonCounter;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ScopeDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog15.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 15 - Bean Scopes Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("\n--- 1. Testing Singleton Scope ---");
            SingletonCounter s1 = context.getBean(SingletonCounter.class);
            SingletonCounter s2 = context.getBean(SingletonCounter.class);
            System.out.println("s1 count: " + s1.increment()); // 1
            System.out.println("s2 count: " + s2.increment()); // 2
            System.out.println("Same Singleton Instance? " + (s1 == s2)); // true

            System.out.println("\n--- 2. Testing Prototype Scope ---");
            PrototypeTask p1 = context.getBean(PrototypeTask.class);
            PrototypeTask p2 = context.getBean(PrototypeTask.class);
            System.out.println("Same Prototype Instance? " + (p1 == p2)); // false

            System.out.println("\nClosing ApplicationContext...");
        }
        System.out.println("Context closed. Notice @PreDestroy runs for Singleton, but NOT Prototype!");
    }
}
```

Expected Output:

```text
=== Blog 15 - Bean Scopes Demo ===

--- 1. Testing Singleton Scope ---
s1 count: 1
s2 count: 2
Same Singleton Instance? true

--- 2. Testing Prototype Scope ---
[PrototypeTask] Instance constructed!
[PrototypeTask] Instance constructed!
Same Prototype Instance? false

Closing ApplicationContext...
Context closed. Notice @PreDestroy runs for Singleton, but NOT Prototype!
```

---

## 6. Key Takeaways

1. `singleton` is the default scope—one instance per `ApplicationContext`.
2. `prototype` creates a new instance on every lookup/injection.
3. Spring does **not** execute destruction callbacks (`@PreDestroy`) for `prototype` beans.

---

## 7. Next Blog

**Part 16: Method Injection and `@Lookup`**
We solve the problem of injecting a short-lived `prototype` bean into a long-lived `singleton` bean using `@Lookup`.

Where you are: **Blog 15 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
