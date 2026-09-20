---
title: "Collection, Generic, and Optional Injection; @Order and @Priority"
description: "Part 13 of the Spring Core series. How to inject List, Set, Map, and generic types, handle optional dependencies with Optional and @Nullable, and order collection items with @Order."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 13

tags:
  - java
  - spring
  - spring-core
  - collection-injection
  - order

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/collection-generic-and-optional-injection-order-and-priority/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 13 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | List/Set/Map autowiring, generic type matching, Optional and @Nullable injection, @Order vs single-bean selection |
| **Concepts unlocked** | Part 17 (ObjectProvider), Part 37 (Type Conversion) |

## 1. Prerequisites

**Spring concepts required**
- Autowiring, `@Primary`, and `@Qualifier` (Parts 10 & 12)

**Previous blogs required**
- Part 10 (*Autowiring and Collaborators*)
- Part 12 (*@Primary and @Qualifier*)

---

## 2. What You Will Learn

- How to inject all matching beans of an interface into a `List<T>`, `Set<T>`, or `Map<String, T>`
- How Spring resolves generic types (e.g. `Repository<Order>` vs `Repository<User>`)
- How to handle optional dependencies cleanly with `Optional<T>`, `@Nullable`, or `required = false`
- The purpose of `@Order` and `Ordered` (Hint: It orders items *inside injected collections*, NOT single bean selection!)

---

## 3. Injecting Collections of Beans

When multiple beans implement the same interface (e.g. `NotificationChannel`), Spring can inject all of them into a collection:

- **`List<T>`**: Injects all instances matching type `T`.
- **`Set<T>`**: Injects unique instances matching type `T`.
- **`Map<String, T>`**: Injects all instances matching type `T`, keyed by their **bean name**.

---

## 4. Ordering Collection Beans (`@Order`)

By default, the order of beans in an injected `List<T>` is non-deterministic. Placing `@Order(n)` on component classes sorts elements in the injected `List<T>` in ascending order (`@Order(1)` comes before `@Order(2)`).

> **CRITICAL RULE**: `@Order` does **NOT** select which single bean gets injected into a non-collection dependency. Use `@Primary` or `@Qualifier` for single bean selection!

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog13/domain/NotificationChannel.java`

```java
package com.example.springcore.blog13.domain;

public interface NotificationChannel {
    void notifyUser(String message);
}
```

**File:** `src/main/java/com/example/springcore/blog13/domain/EmailChannel.java`

```java
package com.example.springcore.blog13.domain;

import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(1) // Executed first in collection
public class EmailChannel implements NotificationChannel {
    @Override
    public void notifyUser(String message) {
        System.out.println("[EmailChannel (Order 1)] Sending email: " + message);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog13/domain/SmsChannel.java`

```java
package com.example.springcore.blog13.domain;

import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(2) // Executed second in collection
public class SmsChannel implements NotificationChannel {
    @Override
    public void notifyUser(String message) {
        System.out.println("[SmsChannel (Order 2)] Sending SMS: " + message);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog13/domain/NotificationService.java`

```java
package com.example.springcore.blog13.domain;

import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.Optional;

@Component
public class NotificationService {

    private final List<NotificationChannel> channels; // Sorted list of channels
    private final Map<String, NotificationChannel> channelMap; // Keyed by bean name
    private final Optional<String> optionalConfig; // Optional dependency

    public NotificationService(
            List<NotificationChannel> channels,
            Map<String, NotificationChannel> channelMap,
            Optional<String> optionalConfig) {
        this.channels = channels;
        this.channelMap = channelMap;
        this.optionalConfig = optionalConfig;
    }

    public void dispatch(String message) {
        System.out.println("--- Dispatching to ordered list ---");
        channels.forEach(ch -> ch.notifyUser(message));

        System.out.println("--- Map keys (Bean names) ---");
        channelMap.keySet().forEach(beanName -> System.out.println("Bean Name: " + beanName));

        System.out.println("Optional Config present? " + optionalConfig.isPresent());
    }
}
```

**File:** `src/main/java/com/example/springcore/blog13/CollectionDemoApp.java`

```java
package com.example.springcore.blog13;

import com.example.springcore.blog13.domain.NotificationService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class CollectionDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog13.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 13 - Collection & Optional Injection Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            NotificationService service = context.getBean(NotificationService.class);
            service.dispatch("Order ORD-888 Confirmed!");
        }
    }
}
```

Expected Output:

```text
=== Blog 13 - Collection & Optional Injection Demo ===
--- Dispatching to ordered list ---
[EmailChannel (Order 1)] Sending email: Order ORD-888 Confirmed!
[SmsChannel (Order 2)] Sending SMS: Order ORD-888 Confirmed!
--- Map keys (Bean names) ---
Bean Name: emailChannel
Bean Name: smsChannel
Optional Config present? false
```

---

## 6. Key Takeaways

1. Autowiring `List<T>` or `Set<T>` collects all beans implementing interface `T`.
2. Autowiring `Map<String, T>` maps bean names to their corresponding instances.
3. `@Order(n)` orders elements inside an injected collection, but does NOT select single beans.
4. `Optional<T>` or `@Nullable` handles optional beans without throwing `NoSuchBeanDefinitionException`.

---

## 7. Next Blog

**Part 14: Circular Dependencies**
We analyze circular dependency cycles (`A → B → A`), why constructor cycles fail fast, and how Spring's three-level cache resolves setter/field cycles.

Where you are: **Blog 13 of 49** (Phase 2: Dependency Resolution and Wiring).
