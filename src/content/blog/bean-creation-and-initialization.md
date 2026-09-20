---
title: "Bean Creation and Initialization: Callbacks and Ordering"
description: "Part 8 of the Spring Core series. Detailed breakdown of constructor execution, property population, Aware callbacks, @PostConstruct, InitializingBean, and custom init-method ordering."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 8

tags:
  - java
  - spring
  - spring-core
  - bean-lifecycle
  - initialization

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/bean-creation-and-initialization/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 8 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 45 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Bean creation phases, constructor invocation, property population, @PostConstruct, InitializingBean, custom init-method |
| **Concepts unlocked** | Part 19 (Complete Bean Lifecycle), Part 22 (BeanPostProcessor) |

## 1. Prerequisites

**Spring concepts required**
- Beans, `BeanDefinition`, and Aware interfaces (Parts 3 & 7)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 7 (*Aware Interfaces*)

---

## 2. What You Will Learn

- The 6 distinct phases of bean creation and initialization
- Why constructor execution is separate from initialization callbacks
- The 3 initialization styles: `@PostConstruct`, `InitializingBean.afterPropertiesSet()`, and `init-method`
- The exact execution order when multiple initialization mechanisms are combined on one bean
- Why accessing dependencies inside a constructor can lead to `NullPointerException`s

---

## 3. The Initialization Phase Sequence

When Spring creates a bean instance, it executes callbacks in this precise order:

```text
1. Constructor Invocation (Raw Object Allocation)
   ↓
2. Property & Dependency Injection
   ↓
3. Aware Callbacks (BeanNameAware, ApplicationContextAware)
   ↓
4. BeanPostProcessor.postProcessBeforeInitialization()
   ↓
5. @PostConstruct Callback (JSR-250)
   ↓
6. InitializingBean.afterPropertiesSet() (Spring Contract)
   ↓
7. Custom init-method (Configured in XML / Java Config)
   ↓
8. BeanPostProcessor.postProcessAfterInitialization()
```

---

## 4. Three Initialization Callback Styles

### 1. `@PostConstruct` Annotation (Recommended)
Defined in `jakarta.annotation.PostConstruct`. Non-invasive and standard across Java ecosystems.

```java
@PostConstruct
public void init() {
    System.out.println("[@PostConstruct] Initializing resource connections...");
}
```

### 2. `InitializingBean` Interface
Spring interface contract requiring `afterPropertiesSet()`.

```java
public class DatabaseService implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("[InitializingBean] Validating database pool...");
    }
}
```

### 3. Custom `init-method`
Configured externally in Java Config (`@Bean(initMethod = "customInit")`) or XML (`<bean init-method="customInit">`). Keeps class free of Spring imports.

```java
public void customInit() {
    System.out.println("[Custom init-method] External init hook executed.");
}
```

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog08/domain/LifecycleBean.java`

```java
package com.example.springcore.blog08.domain;

import jakarta.annotation.PostConstruct;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.InitializingBean;

public class LifecycleBean implements BeanNameAware, InitializingBean {

    private String name;

    public LifecycleBean() {
        System.out.println("1. Constructor: Allocating raw instance.");
    }

    @Override
    public void setBeanName(String name) {
        this.name = name;
        System.out.println("2. Aware Callback: setBeanName('" + name + "')");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct: JSR-250 annotation callback.");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("4. InitializingBean: afterPropertiesSet() callback.");
    }

    public void customInit() {
        System.out.println("5. Custom initMethod: Configured in @Bean.");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog08/InitializationDemoApp.java`

```java
package com.example.springcore.blog08;

import com.example.springcore.blog08.domain.LifecycleBean;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class InitializationDemoApp {

    @Configuration
    static class Config {
        @Bean(initMethod = "customInit")
        public LifecycleBean lifecycleBean() {
            return new LifecycleBean();
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Blog 08 - Bean Initialization Ordering ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            LifecycleBean bean = context.getBean(LifecycleBean.class);
            System.out.println("Bean ready for business operations!");
        }
    }
}
```

Expected Output:

```text
=== Blog 08 - Bean Initialization Ordering ===
1. Constructor: Allocating raw instance.
2. Aware Callback: setBeanName('lifecycleBean')
3. @PostConstruct: JSR-250 annotation callback.
4. InitializingBean: afterPropertiesSet() callback.
5. Custom initMethod: Configured in @Bean.
Bean ready for business operations!
```

---

## 6. Key Takeaways

1. Never perform heavy resource initialization or dependency-dependent setup inside constructors (dependencies aren't injected yet!).
2. Execution order is: **Constructor ──► Injections ──► Aware ──► `@PostConstruct` ──► `InitializingBean` ──► `init-method`**.
3. Prefer `@PostConstruct` for clean application code, and `init-method` for third-party classes.

---

## 7. Next Blog

**Part 9: Bean Lookup and Bean Naming**
We examine bean naming strategies, aliases, explicit vs generated names, and type-based lookup.

Where you are: **Blog 8 of 49** (Phase 1: IoC and Container Fundamentals - Complete!).
