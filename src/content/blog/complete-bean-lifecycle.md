---
title: "Complete Bean Lifecycle: From Instantiation to Destruction"
description: "Part 19 of the Spring Core series. Comprehensive master guide demonstrating the exact execution sequence of all 12+ initialization and destruction callbacks."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 19

tags:
  - java
  - spring
  - spring-core
  - bean-lifecycle
  - destruction-callbacks

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/complete-bean-lifecycle/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 19 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Complete bean lifecycle timeline, InitializingBean, DisposableBean, @PostConstruct, @PreDestroy, Custom init/destroy methods |
| **Concepts unlocked** | Part 21 (Lifecycle and SmartLifecycle), Part 22 (BeanPostProcessor) |

## 1. Prerequisites

**Spring concepts required**
- Bean creation, initialization callbacks, and Aware interfaces (Parts 3, 7 & 8)

**Previous blogs required**
- Part 7 (*Aware Interfaces*)
- Part 8 (*Bean Creation and Initialization*)

---

## 2. The Complete 12-Step Lifecycle Timeline

```text
======================= INITIALIZATION PHASE =======================
 1. Constructor Invocation (Raw Instantiation)
 2. Property & Dependency Injection
 3. Aware Callbacks (BeanNameAware, ApplicationContextAware, etc.)
 4. BeanPostProcessor.postProcessBeforeInitialization()
 5. @PostConstruct Callback (JSR-250)
 6. InitializingBean.afterPropertiesSet()
 7. Custom init-method
 8. BeanPostProcessor.postProcessAfterInitialization()
========================== USAGE PHASE ============================
 9. Bean Ready for Application Business Operations
======================== DESTRUCTION PHASE =========================
10. @PreDestroy Callback (JSR-250)
11. DisposableBean.destroy()
12. Custom destroy-method
```

---

## 3. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog19/domain/MasterLifecycleBean.java`

```java
package com.example.springcore.blog19.domain;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class MasterLifecycleBean implements BeanNameAware, InitializingBean, DisposableBean {

    public MasterLifecycleBean() {
        System.out.println("1. Constructor: Raw instantiation.");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("2. Aware Callback: setBeanName('" + name + "')");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct: JSR-250 initialization hook.");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("4. InitializingBean: afterPropertiesSet() callback.");
    }

    public void customInit() {
        System.out.println("5. Custom init-method: Specified in @Bean(initMethod=...).");
    }

    public void executeBusinessLogic() {
        System.out.println(">>> 6. USAGE PHASE: Executing business logic! <<<");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("7. @PreDestroy: JSR-250 destruction hook.");
    }

    @Override
    public void destroy() {
        System.out.println("8. DisposableBean: destroy() callback.");
    }

    public void customDestroy() {
        System.out.println("9. Custom destroy-method: Specified in @Bean(destroyMethod=...).");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog19/MasterLifecycleApp.java`

```java
package com.example.springcore.blog19;

import com.example.springcore.blog19.domain.MasterLifecycleBean;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class MasterLifecycleApp {

    @Configuration
    static class Config {
        @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
        public MasterLifecycleBean masterLifecycleBean() {
            return new MasterLifecycleBean();
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Blog 19 - Master Bean Lifecycle Timeline ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            MasterLifecycleBean bean = context.getBean(MasterLifecycleBean.class);
            bean.executeBusinessLogic();

            System.out.println("Closing ApplicationContext...");
        }
        System.out.println("Context closed successfully.");
    }
}
```

Expected Output:

```text
=== Blog 19 - Master Bean Lifecycle Timeline ===
1. Constructor: Raw instantiation.
2. Aware Callback: setBeanName('masterLifecycleBean')
3. @PostConstruct: JSR-250 initialization hook.
4. InitializingBean: afterPropertiesSet() callback.
5. Custom init-method: Specified in @Bean(initMethod=...).
>>> 6. USAGE PHASE: Executing business logic! <<<
Closing ApplicationContext...
7. @PreDestroy: JSR-250 destruction hook.
8. DisposableBean: destroy() callback.
9. Custom destroy-method: Specified in @Bean(destroyMethod=...).
Context closed successfully.
```

---

## 4. Key Takeaways

1. Initialization order: **Constructor ──► Aware Callbacks ──► `@PostConstruct` ──► `InitializingBean` ──► Custom `init-method`**.
2. Destruction order: **`@PreDestroy` ──► `DisposableBean` ──► Custom `destroy-method`**.
3. Destruction callbacks only run when `ApplicationContext.close()` is invoked (or shutdown hook triggered) on singletons.

---

## 5. Next Blog

**Part 20: Lazy Initialization and `@DependsOn`**
We explore `@Lazy` beans, explicit dependency sequencing with `@DependsOn`, and startup vs latency trade-offs.

Where you are: **Blog 19 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
