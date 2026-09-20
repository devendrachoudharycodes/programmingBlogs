---
title: "@Configuration and @Bean Semantics: Full vs. Lite Mode and CGLIB Enhancements"
description: "Part 29 of the Spring Core series. CGLIB proxying of @Configuration classes, proxyBeanMethods, Full vs Lite mode, and static @Bean method mechanics."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 29

tags:
  - java
  - spring
  - spring-core
  - configuration
  - cglib

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/configuration-and-bean-semantics/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 29 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 40 min / 45 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Full vs Lite mode, CGLIB subclass enhancement, proxyBeanMethods, inter-bean method calls, static @Bean methods |
| **Concepts unlocked** | Part 30 (Classpath Scanning), Part 33 (@Import and ImportSelectors) |

## 1. Prerequisites

**Spring concepts required**
- Java configuration, `@Bean` definitions, and CGLIB proxies (Parts 9 & 28)

**Previous blogs required**
- Part 9 (*Bean Lookup and Bean Naming*)
- Part 28 (*Annotation Processing Infrastructure*)

---

## 2. What You Will Learn

- The difference between **Full Mode** (`@Configuration`) and **Lite Mode** (`@Component` with `@Bean`)
- How CGLIB enhances `@Configuration` classes to intercept inter-bean method calls
- Controlling CGLIB proxying using `proxyBeanMethods = false`
- Why `BeanFactoryPostProcessor` beans declared with `@Bean` MUST be `static`

---

## 3. Full vs. Lite Mode

### Full Mode (`@Configuration`)
Spring creates a CGLIB subclass proxy of the configuration class. When one `@Bean` method calls another `@Bean` method directly, CGLIB intercepts the call and returns the existing singleton bean from the container!

### Lite Mode (`proxyBeanMethods = false` or `@Component`)
No CGLIB proxy is generated. Calling another `@Bean` method directly executes plain Java method execution, producing a **new unmanaged object instance** on every call!

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog29/domain/DependencyBean.java`

```java
package com.example.springcore.blog29.domain;

public class DependencyBean {
    public DependencyBean() {
        System.out.println("[DependencyBean] Instantiated!");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog29/javaconfig/FullModeConfig.java`

```java
package com.example.springcore.blog29.javaconfig;

import com.example.springcore.blog29.domain.DependencyBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration // Full Mode (proxyBeanMethods = true default)
public class FullModeConfig {

    @Bean
    public DependencyBean dependencyBean() {
        return new DependencyBean();
    }

    @Bean
    public String clientOne() {
        dependencyBean(); // Intercepted by CGLIB: Returns shared singleton
        return "clientOne";
    }

    @Bean
    public String clientTwo() {
        dependencyBean(); // Intercepted by CGLIB: Returns shared singleton
        return "clientTwo";
    }
}
```

**File:** `src/main/java/com/example/springcore/blog29/ConfigurationSemanticsDemoApp.java`

```java
package com.example.springcore.blog29;

import com.example.springcore.blog29.javaconfig.FullModeConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ConfigurationSemanticsDemoApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 29 - @Configuration Full Mode CGLIB Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(FullModeConfig.class)) {

            System.out.println("Notice DependencyBean constructor executed ONLY ONCE due to CGLIB interception!");
        }
    }
}
```

Expected Output:

```text
=== Blog 29 - @Configuration Full Mode CGLIB Demo ===
[DependencyBean] Instantiated!
Notice DependencyBean constructor executed ONLY ONCE due to CGLIB interception!
```

---

## 5. Key Takeaways

1. Full `@Configuration` classes are proxied with CGLIB to enforce singleton semantics for inter-bean method calls.
2. Setting `proxyBeanMethods = false` disables CGLIB enhancement for faster startup times when inter-bean calls aren't used.
3. Use method parameter injection (`@Bean public Service service(Dependency dep)`) instead of direct `@Bean` method calls.

---

## 6. Next Blog

**Part 30: Classpath Scanning**
We explore `@ComponentScan`, `@Component`, `@Service`, `@Repository`, and `@Controller` stereotypes.

Where you are: **Blog 29 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
