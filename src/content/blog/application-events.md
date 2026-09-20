---
title: "Application Events: Decoupling Components with In-Memory Event Publishing"
description: "Part 40 of the Spring Core series. How to publish and consume in-memory application events using ApplicationEventPublisher and @EventListener."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 40

tags:
  - java
  - spring
  - spring-core
  - application-events
  - event-listener

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/application-events/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 40 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | ApplicationEventPublisher, @EventListener, custom POJO events, synchronous event dispatching |
| **Concepts unlocked** | Part 41 (MessageSource), Part 42 (ApplicationContext Architecture) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` container and `ApplicationEventPublisherAware` (Parts 6 & 7)

**Previous blogs required**
- Part 6 (*BeanFactory vs ApplicationContext*)
- Part 7 (*Aware Interfaces*)

---

## 2. What You Will Learn

- How Spring's in-memory Event Bus decouples publishers from listeners
- Publishing custom POJO events via `ApplicationEventPublisher`
- Listening for events with `@EventListener`
- Understanding synchronous event execution by default
- Built-in container events (`ContextRefreshedEvent`, `ContextClosedEvent`)

---

## 3. Event Execution Architecture

```text
1. Publisher calls publisher.publishEvent(new OrderPlacedEvent("ORD-101"))
   ↓
2. ApplicationContext (ApplicationEventPublisher) receives event
   ↓
3. Finds all @EventListener methods matching event payload type
   ↓
4. Synchronously invokes listeners sequentially on calling thread
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog40/event/OrderPlacedEvent.java`

```java
package com.example.springcore.blog40.event;

public record OrderPlacedEvent(String orderId, double amount) {
}
```

**File:** `src/main/java/com/example/springcore/blog40/listener/NotificationListener.java`

```java
package com.example.springcore.blog40.listener;

import com.example.springcore.blog40.event.OrderPlacedEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class NotificationListener {

    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        System.out.println("[NotificationListener] Order confirmation email sent for " + event.orderId());
    }
}
```

**File:** `src/main/java/com/example/springcore/blog40/domain/OrderService.java`

```java
package com.example.springcore.blog40.domain;

import com.example.springcore.blog40.event.OrderPlacedEvent;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Component
public class OrderService {

    private final ApplicationEventPublisher publisher;

    public OrderService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void placeOrder(String orderId, double amount) {
        System.out.println("[OrderService] Placing order " + orderId);

        // Publish event to decoupled listeners!
        publisher.publishEvent(new OrderPlacedEvent(orderId, amount));
    }
}
```

**File:** `src/main/java/com/example/springcore/blog40/EventDemoApp.java`

```java
package com.example.springcore.blog40;

import com.example.springcore.blog40.domain.OrderService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class EventDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog40")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 40 - Application Events Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            OrderService service = context.getBean(OrderService.class);
            service.placeOrder("ORD-9009", 149.99);
        }
    }
}
```

Expected Output:

```text
=== Blog 40 - Application Events Demo ===
[OrderService] Placing order ORD-9009
[NotificationListener] Order confirmation email sent for ORD-9009
```

---

## 5. Key Takeaways

1. `ApplicationEventPublisher` decouples business domain services from side-effect tasks (emails, audit logs).
2. Modern Spring (4.2+) allows publishing plain Java objects (POJOs) as events without extending `ApplicationEvent`.
3. Event listeners run **synchronously** on the publisher's calling thread by default.

---

## 6. Next Blog

**Part 41: MessageSource and Internationalization**
We explore i18n message resolution, `ResourceBundleMessageSource`, and locale resolution.

Where you are: **Blog 40 of 49** (Phase 4b: Environment and Infrastructure Abstractions).
