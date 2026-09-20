---
title: "Beans and the Spring IoC Container: Definitions, Identity, and Resolution"
description: "Part 3 of the Spring Core series. What makes an object a Spring bean, how BeanDefinition metadata represents bean recipes, and the 10-step resolution process executed by the ApplicationContext."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 3

tags:
  - java
  - spring
  - spring-core
  - spring-beans
  - application-context

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/beans-and-the-spring-ioc-container/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 3 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 30 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | What a bean is, BeanDefinition metadata, BeanFactory vs ApplicationContext responsibilities, the 10-step resolution process |
| **Concepts unlocked** | Part 4 (Building Spring from Scratch with Maven), Part 6 (BeanFactory vs ApplicationContext), Part 26 (BeanDefinition deep dive) |

## 1. Prerequisites

**Java concepts required**
- Object instantiation, reflection basics, interface implementation

**Spring concepts required**
- IoC and Dependency Injection fundamentals (Part 2)

**Previous blogs required**
- Part 1 (*What Is Spring? The Problem, the Container, and the Mental Model*)
- Part 2 (*IoC and DI Fundamentals*)

---

## 2. What You Will Learn

- What defines a **Spring Bean** versus a plain Java object
- The anatomy of a **BeanDefinition** (metadata, class name, scope, constructor args, property values)
- How the **Spring IoC Container** acts as a metadata processor and factory manager
- The **10-step resolution process** from metadata loading to final bean exposure
- The conceptual difference between `BeanFactory` and `ApplicationContext`
- How bean identity, scoping, and naming are tracked inside the container
- Common mistakes when querying the container directly

---

## 3. Why This Topic Matters

In Java, any object instantiated with `new MyObject()` exists on the heap, managed by the JVM garbage collector. However, in Spring applications, not every Java object is a **Spring Bean**.

A Spring Bean is a Java object created, configured, wired, and managed by the Spring IoC container according to metadata recipes (`BeanDefinition`). Understanding how these recipes are registered, processed, and instantiated is key to mastering Spring's lifecycle hooks, proxies, and extension points.

---

## 4. Real-World Problem

Consider a scenario where an application instantiates services manually versus retrieving them as container-managed beans.

**File:** `src/main/java/com/example/springcore/blog03/plain/PlainObjectVsBeanApp.java`

```java
package com.example.springcore.blog03.plain;

import java.math.BigDecimal;

public class PlainObjectVsBeanApp {

    static class PaymentService {
        public void processPayment(BigDecimal amount) {
            System.out.println("Processing payment of $" + amount);
        }
    }

    public static void main(String[] args) {
        // Plain Java Object: JVM manages heap memory, but no container lifecycle or wiring exists
        PaymentService plainService1 = new PaymentService();
        PaymentService plainService2 = new PaymentService();

        System.out.println("Are plain instances identical? " + (plainService1 == plainService2)); // false
    }
}
```

When managed by Spring, the container ensures single instance sharing (by default), automated dependency wiring, lifecycle callback invocation, and optional proxy wrapping.

---

## 5. Core Concept

### 5.1 What is a Spring Bean?

A **Spring Bean** is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Beans are created with configuration metadata that you supply to the container (via XML, Java annotations, or Java configuration code).

Key Attributes of a Bean:
- **Identity**: Unique name/id within the container.
- **Type**: The underlying Java class or interface.
- **Scope**: Lifecycle boundaries (e.g. `singleton`, `prototype`).
- **Dependencies**: Collaborators required for instantiation and initialization.
- **Lifecycle Callbacks**: Pre-initialization and post-destruction methods.

---

### 5.2 The 10-Step Resolution Process

When the Spring container starts and prepares beans for consumption, it executes a rigorous 10-step lifecycle:

```text
 1. Metadata Loading ──► 2. BeanDefinition Registration ──► 3. BeanFactoryPostProcessors
                                                                    │
 6. Property Population ◄── 5. Instantiation ◄── 4. Dependency Resolution
          │
          ▼
 7. Aware Callbacks ──► 8. BeanPostProcessor (Before) ──► 9. Init Callbacks ──► 10. Final Exposure
```

1. **Metadata Loading**: Reads XML files, `@Configuration` classes, or `@Component` annotations.
2. **BeanDefinition Registration**: Parses metadata into `BeanDefinition` objects stored in a registry map (`BeanDefinitionRegistry`).
3. **BeanFactoryPostProcessor Execution**: Modifies metadata recipes before any objects are created (e.g., property placeholder substitution `${db.url}`).
4. **Dependency Resolution**: Analyzes required constructor arguments and setter properties for required dependencies.
5. **Instantiation**: Creates the raw Java object instance using constructor or factory method reflection.
6. **Property Population**: Injects dependencies into fields and invokes setters.
7. **Aware Callbacks**: Injects container references if the bean implements Aware interfaces (`BeanNameAware`, `ApplicationContextAware`).
8. **BeanPostProcessor (Before Initialization)**: Executes `postProcessBeforeInitialization` on custom post-processors.
9. **Initialization Callbacks**: Invokes `@PostConstruct`, `InitializingBean.afterPropertiesSet()`, or custom `init-method`.
10. **Final Bean Exposure & Wrapping**: Executes `postProcessAfterInitialization` (creating AOP proxies if applicable) and registers the singleton instance for application use.

---

## 6. Mental Model

```text
       METADATA                     CONTAINER                     APPLICATION
  ┌─────────────────┐       ┌───────────────────────┐       ┌─────────────────────┐
  │ XML / Java /    │ ───►  │ BeanDefinition Map    │ ───►  │ Managed Bean        │
  │ @Component      │       │ (Recipes)             │       │ Instances           │
  └─────────────────┘       └───────────────────────┘       └─────────────────────┘
```

The container stores **recipes** (`BeanDefinition`), not objects, during startup. It builds **instances** on demand or during eager pre-instantiation.

---

## 7. Configuration Examples

### 7.1 XML Configuration Style

**File:** `src/main/resources/blog03/beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="paymentService" class="com.example.springcore.blog03.domain.PaymentService"/>

</beans>
```

### 7.2 Java Configuration Style

**File:** `src/main/java/com/example/springcore/blog03/javaconfig/JavaAppConfig.java`

```java
package com.example.springcore.blog03.javaconfig;

import com.example.springcore.blog03.domain.PaymentService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaAppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

### 7.3 Annotation Configuration Style

**File:** `src/main/java/com/example/springcore/blog03/annotation/PaymentService.java`

```java
package com.example.springcore.blog03.annotation;

import org.springframework.stereotype.Component;

@Component
public class PaymentService {
    public void pay() {
        System.out.println("[PaymentService] Executing payment...");
    }
}
```

---

## 8. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog03/domain/PaymentService.java`

```java
package com.example.springcore.blog03.domain;

public class PaymentService {
    public void pay() {
        System.out.println("[PaymentService] Executing payment...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog03/javaconfig/BeanDemoApp.java`

```java
package com.example.springcore.blog03.javaconfig;

import com.example.springcore.blog03.domain.PaymentService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class BeanDemoApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 03 - Bean Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(JavaAppConfig.class)) {

            // Retrieve managed bean
            PaymentService service1 = context.getBean(PaymentService.class);
            PaymentService service2 = context.getBean("paymentService", PaymentService.class);

            service1.pay();

            // Singletons share identity
            System.out.println("Are bean instances identical? " + (service1 == service2)); // true
        }
    }
}
```

---

## 9. Common Mistakes

1. **Expecting `new`ed objects to be Spring Beans**: Manually instantiated objects bypass the 10-step resolution process.
2. **Confusing Bean Name with Variable Name**: Bean names default to class name with lowercase initial or method name in Java config.
3. **Assuming Beans exist before `ApplicationContext` refresh**: Registrations must complete before instances exist.

---

## 10. Interview Questions

### Beginner
1. **What is a Spring Bean?**
   An object managed by the Spring IoC container created from metadata.
2. **How does a Spring Bean differ from a plain Java object?**
   Spring Beans undergo container lifecycle management, autowiring, and proxying.

### Intermediate
1. **What is a `BeanDefinition`?**
   A metadata representation (recipe) describing class name, scope, properties, and constructor arguments for a bean.
2. **What are the 2 main container interfaces?**
   `BeanFactory` (basic container) and `ApplicationContext` (enterprise-ready extension).

---

## 11. Key Takeaways

1. Beans are metadata-driven objects managed by the container.
2. `BeanDefinition` recipes precede bean instances.
3. Singletons are shared across the container lifetime by default.

---

## 12. Next Blog

**Part 4: Building Spring from Scratch with Maven**
We step back to build a clean Spring project from an empty directory, detailing every line of `pom.xml` and dependencies.

Where you are: **Blog 3 of 49** (Phase 1: IoC and Container Fundamentals).
