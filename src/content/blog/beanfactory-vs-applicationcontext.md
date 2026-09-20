---
title: "BeanFactory vs ApplicationContext: Core Container vs Enterprise Framework"
description: "Part 6 of the Spring Core series. Deep comparison of BeanFactory and ApplicationContext, lazy vs eager initialization, and why ApplicationContext is the standard container for modern applications."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 6

tags:
  - java
  - spring
  - spring-core
  - bean-factory
  - application-context

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/beanfactory-vs-applicationcontext/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 6 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 30 min / ~60 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | BeanFactory interface, ApplicationContext interface, lazy vs eager initialization, enterprise services (events, i18n, resources) |
| **Concepts unlocked** | Part 7 (Aware Interfaces), Part 40 (Application Events), Part 42 (ApplicationContext Internal Architecture) |

## 1. Prerequisites

**Spring concepts required**
- Beans and container basics (Parts 1–3)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 4 (*Building Spring from Scratch with Maven*)

---

## 2. What You Will Learn

- The hierarchy relationship between `BeanFactory` and `ApplicationContext`
- Why `BeanFactory` is a lightweight lazy container and `ApplicationContext` is an eager enterprise container
- Additional capabilities provided by `ApplicationContext` (Event publishing, Resource loading, i18n, Environment profiles)
- Why using a raw `DefaultListableBeanFactory` requires manual post-processor registration
- Direct comparison table between `BeanFactory` and `ApplicationContext`

---

## 3. Core Hierarchy Comparison

`BeanFactory` is the root interface of the Spring IoC container, providing basic bean lookup and instantiation capabilities. `ApplicationContext` extends `BeanFactory` and adds enterprise-grade framework features.

```text
                       ┌────────────────────┐
                       │    BeanFactory     │  (Basic Container: getBean, containsBean)
                       └─────────┬──────────┘
                                 │
                       ┌─────────┴──────────┐
                       │ ListableBeanFactory│
                       └─────────┬──────────┘
                                 │
┌────────────────────────────────┴─────────────────────────────────┐
│                       ApplicationContext                         │
├──────────────────────────────────────────────────────────────────┤
│ + EnvironmentCapable       (Profiles, Property Sources)          │
│ + MessageSource            (i18n, Internationalization)          │
│ + ApplicationEventPublisher(In-memory Event Bus)                 │
│ + ResourcePatternResolver  (Classpath / File Resource Loading)   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Key Differences

### 1. Bean Initialization Timing
- **`BeanFactory`**: Beans are initialized **lazily** on first `getBean()` call.
- **`ApplicationContext`**: Singleton beans are pre-instantiated **eagerly** during startup (`refresh()` phase). Uncovers wiring errors immediately on startup.

### 2. Post-Processor Registration
- **`BeanFactory`**: `BeanFactoryPostProcessor` and `BeanPostProcessor` implementations must be registered manually in code.
- **`ApplicationContext`**: Automatically detects and registers all post-processor beans.

---

## 5. Direct Comparison Table

| Capability / Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| **Bean Instantiation** | Lazy (on demand via `getBean`) | Eager (for singletons at startup) |
| **Automatic Post-Processor Registration** | ❌ No (Manual registration required) | ✅ Yes (Automatic detection) |
| **Application Events (`ApplicationEvent`)** | ❌ No | ✅ Yes (`ApplicationEventPublisher`) |
| **MessageSource (i18n)** | ❌ No | ✅ Yes |
| **Environment & Profile Support** | ❌ Limited | ✅ Yes (`EnvironmentCapable`) |
| **Resource Pattern Resolution** | ❌ Basic | ✅ Full (`ResourcePatternResolver`) |
| **AOP & Declarative Annotation Support** | ❌ Requires manual wiring | ✅ Full out-of-the-box support |
| **Memory Footprint** | Extremely low | Slightly higher (startup overhead) |
| **Modern Usage** | Embedded/Memory-constrained devices | Standard for all enterprise applications |

---

## 6. Complete Working Example

This example demonstrates using raw `DefaultListableBeanFactory` versus `ApplicationConfigApplicationContext`.

**File:** `src/main/java/com/example/springcore/blog06/domain/SampleBean.java`

```java
package com.example.springcore.blog06.domain;

public class SampleBean {
    public SampleBean() {
        System.out.println("[SampleBean] Constructor executed!");
    }

    public void doWork() {
        System.out.println("[SampleBean] Doing work...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog06/BeanFactoryVsContextApp.java`

```java
package com.example.springcore.blog06;

import com.example.springcore.blog06.domain.SampleBean;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class BeanFactoryVsContextApp {

    @Configuration
    static class Config {
        @Bean
        public SampleBean sampleBean() {
            return new SampleBean();
        }
    }

    public static void main(String[] args) {
        System.out.println("=== 1. Testing Raw BeanFactory (Lazy Instantiation) ===");
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        factory.registerBeanDefinition("sampleBean", new RootBeanDefinition(SampleBean.class));
        System.out.println("BeanFactory created. Notice SampleBean constructor has NOT run yet!");

        System.out.println("Calling factory.getBean()...");
        SampleBean b1 = factory.getBean(SampleBean.class);
        b1.doWork();

        System.out.println("\n=== 2. Testing ApplicationContext (Eager Instantiation) ===");
        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {
            System.out.println("ApplicationContext created. Notice SampleBean constructor ran during refresh!");
            SampleBean b2 = context.getBean(SampleBean.class);
            b2.doWork();
        }
    }
}
```

Expected Output:

```text
=== 1. Testing Raw BeanFactory (Lazy Instantiation) ===
BeanFactory created. Notice SampleBean constructor has NOT run yet!
Calling factory.getBean()...
[SampleBean] Constructor executed!
[SampleBean] Doing work...

=== 2. Testing ApplicationContext (Eager Instantiation) ===
[SampleBean] Constructor executed!
ApplicationContext created. Notice SampleBean constructor ran during refresh!
[SampleBean] Doing work...
```

---

## 7. Key Takeaways

1. `BeanFactory` is the low-level container SPI; `ApplicationContext` is the high-level application API.
2. `ApplicationContext` pre-instantiates singletons at startup, detecting configuration errors early.
3. `ApplicationContext` adds events, i18n, environment profiles, and automatic post-processor detection.
4. Modern Spring development always uses `ApplicationContext`.

---

## 8. Next Blog

**Part 7: Aware Interfaces**
We explore `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware`, and how beans interact with the container infrastructure when necessary.

Where you are: **Blog 6 of 49** (Phase 1: IoC and Container Fundamentals).
