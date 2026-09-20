---
title: "Using XML, Java Configuration, and Annotations Together"
description: "Part 5 of the Spring Core series. How to compose hybrid Spring configurations using @ImportResource, @Import, and <context:component-scan>, and how to handle bean collisions and precedence."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 5

tags:
  - java
  - spring
  - spring-core
  - configuration
  - hybrid-config

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/using-xml-java-configuration-and-annotations-together/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 5 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 50 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Composition of hybrid configuration styles, @ImportResource, @Import, <context:component-scan>, bean definition overriding and precedence |
| **Concepts unlocked** | Part 29 (@Configuration and @Bean Semantics), Part 33 (@Import and ImportSelectors) |

## 1. Prerequisites

**Java concepts required**
- Annotations, Java class imports, packages

**Spring concepts required**
- XML, Java, and Annotation configuration basics (Parts 1–4)

**Previous blogs required**
- Part 4 (*Building Spring from Scratch with Maven*)

---

## 2. What You Will Learn

- How XML, Java `@Configuration`, and `@Component` annotations coexist in enterprise applications
- How to import XML definitions into Java Configuration using `@ImportResource`
- How to compose multiple Java configuration classes using `@Import`
- How to trigger component scanning from XML using `<context:component-scan>`
- How Spring resolves bean name collisions, duplicate definitions, and overriding precedence
- Recommended hybrid configuration architecture for modern migration projects

---

## 3. Real-World Problem

In real enterprise projects, you rarely start with 100% brand-new code. You often inherit legacy XML bean definitions (e.g. for database pools or legacy messaging), third-party libraries requiring Java `@Configuration` beans, and new domain classes using `@Component` annotations.

Knowing how to compose these three styles into a single cohesive `ApplicationContext` is critical for refactoring legacy systems without breaking existing wiring.

---

## 4. Hybrid Configuration Mechanics

### 4.1 Importing XML into Java Configuration (`@ImportResource`)

```java
@Configuration
@ImportResource("classpath:blog05/legacy-beans.xml")
public class AppConfig {
    @Bean
    public ModernService modernService(LegacyBean legacyBean) {
        return new ModernService(legacyBean);
    }
}
```

### 4.2 Importing Java Config into Java Config (`@Import`)

```java
@Configuration
@Import({ DatabaseConfig.class, SecurityConfig.class })
public class MainConfig {
}
```

### 4.3 Component Scanning from XML (`<context:component-scan>`)

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.example.springcore.blog05.annotation"/>
</beans>
```

---

## 5. Bean Definition Overriding and Precedence

When multiple configuration sources define a bean with the exact same ID (e.g., `paymentGateway`), Spring follows deterministic resolution rules:

1. **Explicit `@Bean` / XML definitions override `@Component` scanned beans**.
2. **Later loaded configuration definitions override earlier definitions** (when bean overriding is enabled).
3. In plain Spring Framework, bean overriding is **allowed by default**. (In Spring Boot, it is disabled by default to prevent silent overrides).

---

## 6. Complete Working Example

**File:** `src/main/resources/blog05/legacy-beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="legacyGateway" class="com.example.springcore.blog05.domain.LegacyGateway"/>
</beans>
```

**File:** `src/main/java/com/example/springcore/blog05/domain/LegacyGateway.java`

```java
package com.example.springcore.blog05.domain;

public class LegacyGateway {
    public void execute() {
        System.out.println("[LegacyGateway] Executed via XML configuration");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog05/annotation/AnnotatedService.java`

```java
package com.example.springcore.blog05.annotation;

import org.springframework.stereotype.Component;

@Component
public class AnnotatedService {
    public void execute() {
        System.out.println("[AnnotatedService] Executed via Component Scanning");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog05/javaconfig/HybridAppConfig.java`

```java
package com.example.springcore.blog05.javaconfig;

import com.example.springcore.blog05.annotation.AnnotatedService;
import com.example.springcore.blog05.domain.LegacyGateway;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;

@Configuration
@ComponentScan(basePackages = "com.example.springcore.blog05.annotation")
@ImportResource("classpath:blog05/legacy-beans.xml")
public class HybridAppConfig {

    @Bean
    public String hybridBean(LegacyGateway legacyGateway, AnnotatedService annotatedService) {
        System.out.println("=== Initializing Hybrid Application ===");
        legacyGateway.execute();
        annotatedService.execute();
        return "HybridReady";
    }
}
```

**File:** `src/main/java/com/example/springcore/blog05/HybridConfigApp.java`

```java
package com.example.springcore.blog05;

import com.example.springcore.blog05.javaconfig.HybridAppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HybridConfigApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 05 - Hybrid Configuration Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(HybridAppConfig.class)) {

            String status = context.getBean("hybridBean", String.class);
            System.out.println("Status: " + status);
        }
    }
}
```

Expected Output:

```text
=== Blog 05 - Hybrid Configuration Demo ===
=== Initializing Hybrid Application ===
[LegacyGateway] Executed via XML configuration
[AnnotatedService] Executed via Component Scanning
Status: HybridReady
```

---

## 7. Key Takeaways

1. Use `@ImportResource` to bring existing XML files into Java Configuration.
2. Use `@Import` to combine modular `@Configuration` classes.
3. Use `@ComponentScan` for application-owned domain components.
4. `@Bean` and XML definitions take precedence over scanned `@Component` beans.

---

## 8. Next Blog

**Part 6: BeanFactory vs ApplicationContext**
We compare Spring's fundamental `BeanFactory` container with the enterprise `ApplicationContext`, examining lazy vs eager initialization and enterprise services.

Where you are: **Blog 5 of 49** (Phase 1: IoC and Container Fundamentals).
