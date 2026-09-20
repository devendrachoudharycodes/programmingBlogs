---
title: "BeanDefinition: Metadata Recipes and Internal Convergence"
description: "Part 26 of the Spring Core series. Deep dive into org.springframework.beans.factory.config.BeanDefinition metadata recipes and how XML, Java Config, and Annotations converge."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 26

tags:
  - java
  - spring
  - spring-core
  - bean-definition
  - internals

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/beandefinition/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 26 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 45 min / 50 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | BeanDefinition API, metadata properties, RootBeanDefinition, GenericBeanDefinition, metadata convergence across XML/Java/Annotations |
| **Concepts unlocked** | Part 27 (Programmatic Registration), Part 28 (Annotation Processing Infrastructure) |

## 1. Prerequisites

**Spring concepts required**
- Beans, container extension points, and `FactoryBean` (Parts 3, 23 & 25)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 23 (*Container Extension Points*)

---

## 2. What You Will Learn

- What `BeanDefinition` is and why it is the core data structure of Spring
- Anatomy of `BeanDefinition` (Bean Class, Scope, Lazy Init, Constructor Arguments, Property Values, Init/Destroy Methods, Role)
- How XML, Java `@Configuration`, and `@Component` scanning all converge into identical `BeanDefinition` recipes
- Concrete implementation classes: `GenericBeanDefinition`, `RootBeanDefinition`, and `AnnotatedGenericBeanDefinition` *(implementation details)*

---

## 3. Metadata Convergence

No matter how you configure Spring, all configuration sources are parsed into `BeanDefinition` instances during startup before any application objects are constructed.

```text
  XML Metadata (<bean>)           ──► XmlBeanDefinitionReader
  Java Config (@Bean)             ──► ConfigurationClassPostProcessor ──► BeanDefinitionRegistry
  Annotations (@Component)        ──► ClasspathBeanDefinitionScanner
```

---

## 4. `BeanDefinition` Properties

```java
public interface BeanDefinition {
    String getBeanClassName();
    String getScope();
    boolean isLazyInit();
    ConstructorArgumentValues getConstructorArgumentValues();
    MutablePropertyValues getPropertyValues();
    String getInitMethodName();
    String getDestroyMethodName();
    int getRole(); // ROLE_APPLICATION, ROLE_SUPPORT, ROLE_INFRASTRUCTURE
    boolean isPrimary();
}
```

---

## 5. Complete Working Example (Inspecting `BeanDefinition`s)

**File:** `src/main/java/com/example/springcore/blog26/domain/SampleService.java`

```java
package com.example.springcore.blog26.domain;

import org.springframework.context.annotation.Lazy;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component("sampleService")
@Scope("singleton")
@Lazy
public class SampleService {
}
```

**File:** `src/main/java/com/example/springcore/blog26/BeanDefinitionInspectorApp.java`

```java
package com.example.springcore.blog26;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class BeanDefinitionInspectorApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog26.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 26 - BeanDefinition Inspection Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            // Retrieve internal BeanDefinition recipe
            BeanDefinition bd = context.getBeanDefinition("sampleService");

            System.out.println("Bean Class Name : " + bd.getBeanClassName());
            System.out.println("Bean Scope      : " + bd.getScope());
            System.out.println("Is Lazy Init?   : " + bd.isLazyInit());
            System.out.println("Is Primary?     : " + bd.isPrimary());
            System.out.println("Bean Role       : " + bd.getRole() + " (ROLE_APPLICATION)");
            System.out.println("Implementation  : " + bd.getClass().getName());
        }
    }
}
```

Expected Output:

```text
=== Blog 26 - BeanDefinition Inspection Demo ===
Bean Class Name : com.example.springcore.blog26.domain.SampleService
Bean Scope      : singleton
Is Lazy Init?   : true
Is Primary?     : false
Bean Role       : 0 (ROLE_APPLICATION)
Implementation  : org.springframework.context.annotation.ScannedGenericBeanDefinition
```

---

## 6. Key Takeaways

1. `BeanDefinition` is the internal blueprint recipe for creating beans.
2. XML, Java Config, and Annotation scanning all converge into `BeanDefinition` objects in `BeanDefinitionRegistry`.
3. Holding metadata recipes as data allows Spring to validate, order, edit, and wrap beans before instantiation.

---

## 7. Next Blog

**Part 27: Programmatic Registration and `BeanDefinitionRegistry`**
We explore programmatic bean registration using `BeanDefinitionRegistry`, `BeanDefinitionBuilder`, and Supplier-based registration.

Where you are: **Blog 26 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
