---
title: "BeanPostProcessor: Intercepting, Modifying, and Wrapping Beans"
description: "Part 22 of the Spring Core series. Deep dive into org.springframework.beans.factory.config.BeanPostProcessor, lifecycle hooks, and building custom bean transformers."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 22

tags:
  - java
  - spring
  - spring-core
  - bean-post-processor
  - extensibility

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/beanpostprocessor/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 22 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 40 min / 55 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | BeanPostProcessor API, postProcessBeforeInitialization, postProcessAfterInitialization, bean mutation & wrapping, PriorityOrdered |
| **Concepts unlocked** | Part 23 (Container Extension Points), Part 24 (How Spring AOP Uses BeanPostProcessor) |

## 1. Prerequisites

**Spring concepts required**
- Complete bean lifecycle timeline (Part 19)

**Previous blogs required**
- Part 8 (*Bean Creation and Initialization*)
- Part 19 (*Complete Bean Lifecycle*)

---

## 2. What You Will Learn

- What a `BeanPostProcessor` (BPP) is and why it is the engine of Spring's extensibility
- The exact execution hooks: `postProcessBeforeInitialization` and `postProcessAfterInitialization`
- How BPPs can inspect, mutate, or wrap bean instances into dynamic proxies
- How BPP ordering is controlled via `PriorityOrdered` and `Ordered`
- Building a custom audit logging `BeanPostProcessor`

---

## 3. `BeanPostProcessor` Execution Sequence

```text
1. Bean Instantiated & Properties Injected
   ↓
2. BeanPostProcessor.postProcessBeforeInitialization(bean, beanName)
   ↓
3. @PostConstruct / InitializingBean / init-method Callbacks
   ↓
4. BeanPostProcessor.postProcessAfterInitialization(bean, beanName)
   ↓
5. Final Bean Instance Exposed to Container
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog22/domain/AuditLoggerBpp.java`

```java
package com.example.springcore.blog22.domain;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.core.PriorityOrdered;
import org.springframework.stereotype.Component;

@Component
public class AuditLoggerBpp implements BeanPostProcessor, PriorityOrdered {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (!beanName.contains(".")) { // Filter out internal infrastructure beans
            System.out.println("[BPP Before Init] Inspecting bean: " + beanName);
        }
        return bean; // Return original or modified bean
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!beanName.contains(".")) {
            System.out.println("[BPP After Init] Bean initialized & wrapped: " + beanName);
        }
        return bean;
    }

    @Override
    public int getOrder() {
        return HIGHEST_PRECEDENCE; // Order execution
    }
}
```

**File:** `src/main/java/com/example/springcore/blog22/domain/PaymentService.java`

```java
package com.example.springcore.blog22.domain;

import jakarta.annotation.PostConstruct;
import org.springframework.stereotype.Component;

@Component
public class PaymentService {

    public PaymentService() {
        System.out.println("-> Constructor: Instantiating PaymentService");
    }

    @PostConstruct
    public void init() {
        System.out.println("-> @PostConstruct: Initializing PaymentService");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog22/BppDemoApp.java`

```java
package com.example.springcore.blog22;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class BppDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog22.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 22 - BeanPostProcessor Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("Context ready!");
        }
    }
}
```

Expected Output:

```text
=== Blog 22 - BeanPostProcessor Demo ===
-> Constructor: Instantiating PaymentService
[BPP Before Init] Inspecting bean: paymentService
-> @PostConstruct: Initializing PaymentService
[BPP After Init] Bean initialized & wrapped: paymentService
Context ready!
```

---

## 5. Key Takeaways

1. `BeanPostProcessor` hooks execute for **every** bean created in the container.
2. `postProcessBeforeInitialization` runs before `@PostConstruct` / `init-method`.
3. `postProcessAfterInitialization` runs after init callbacks and is where Spring AOP creates proxies.

---

## 6. Next Blog

**Part 23: Container Extension Points**
We compare `BeanPostProcessor`, `BeanFactoryPostProcessor`, `BeanDefinitionRegistryPostProcessor`, and `FactoryBean`.

Where you are: **Blog 22 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
