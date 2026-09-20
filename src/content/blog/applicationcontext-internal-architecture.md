---
title: "ApplicationContext Internal Architecture: AbstractApplicationContext and Delegation"
description: "Part 42 of the Spring Core series. Inside AbstractApplicationContext and how it delegates bean factory, event, and resource operations."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 42

tags:
  - java
  - spring
  - spring-core
  - application-context
  - internals

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/applicationcontext-internal-architecture/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 42 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 40 min / 20 min / ~60 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | AbstractApplicationContext hierarchy, delegation to DefaultListableBeanFactory, internal subsystem composition |
| **Concepts unlocked** | Part 43 (ApplicationContext Implementations), Part 45 (Refresh Process) |

## 1. Prerequisites

**Spring concepts required**
- `BeanFactory`, `ApplicationContext`, Events, and Resources (Parts 6, 39 & 40)

**Previous blogs required**
- Part 6 (*BeanFactory vs ApplicationContext*)
- Part 40 (*Application Events*)

---

## 2. What You Will Learn

- How `AbstractApplicationContext` acts as the master coordinator
- How `ApplicationContext` delegates bean operations to an internal `DefaultListableBeanFactory` *(implementation detail)*
- How event publishing, resource pattern matching, and message resolution delegates operate under the hood

---

## 3. Internal Subsystem Delegation Model

```text
                        ┌───────────────────────────────┐
                        │   AbstractApplicationContext  │
                        └───────────────┬───────────────┘
                                        │ Delegates to:
          ┌─────────────────────────────┼─────────────────────────────┐
          ▼                             ▼                             ▼
┌──────────────────┐          ┌───────────────────┐         ┌───────────────────┐
│ DefaultListable- │          │ SimpleApplication-│         │ ResourcePattern-  │
│ BeanFactory      │          │ EventMulticaster  │         │ Resolver          │
│ (Bean Storage)   │          │ (Events)          │         │ (File & IO)       │
└──────────────────┘          └───────────────────┘         └───────────────────┘
```

When you call `context.getBean("myBean")`, `AbstractApplicationContext` forwards the call directly to its internal `DefaultListableBeanFactory` instance.

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog42/ArchitectureInspectorApp.java`

```java
package com.example.springcore.blog42;

import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

public class ArchitectureInspectorApp {

    @Configuration
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 42 - ApplicationContext Architecture Inspection ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            // Inspect underlying BeanFactory delegate
            DefaultListableBeanFactory factory = context.getDefaultListableBeanFactory();

            System.out.println("Context Class        : " + context.getClass().getName());
            System.out.println("Underlying Factory   : " + factory.getClass().getName());
            System.out.println("Factory Bean Count   : " + factory.getBeanDefinitionCount());
            System.out.println("Allow Bean Overriding: " + factory.isAllowBeanDefinitionOverriding());
        }
    }
}
```

Expected Output:

```text
=== Blog 42 - ApplicationContext Architecture Inspection ===
Context Class        : org.springframework.context.annotation.AnnotationConfigApplicationContext
Underlying Factory   : org.springframework.beans.factory.support.DefaultListableBeanFactory
Factory Bean Count   : 6
Allow Bean Overriding: true
```

---

## 5. Key Takeaways

1. `AbstractApplicationContext` provides the template implementation for container refresh, startup, and shutdown.
2. It delegates bean management directly to `DefaultListableBeanFactory`.
3. Event multicasting and resource loading are composed as separate internal delegates.

---

## 6. Next Blog

**Part 43: `ApplicationContext` Implementations**
We compare `AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext`, `FileSystemXmlApplicationContext`, and web contexts.

Where you are: **Blog 42 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
