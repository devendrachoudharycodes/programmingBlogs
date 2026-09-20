---
title: "@Value and Property Resolution: Placeholders and Default Values"
description: "Part 36 of the Spring Core series. Injecting environment properties with @Value, property placeholder resolution (${...}), default values, and SpEL preview."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 36

tags:
  - java
  - spring
  - spring-core
  - value-annotation
  - property-resolution

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/value-and-property-resolution/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 36 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | @Value annotation, ${...} property placeholders, default values (${key:default}), SpEL #{...} evaluation |
| **Concepts unlocked** | Part 37 (Type Conversion), Part 38 (SpEL Fundamentals) |

## 1. Prerequisites

**Spring concepts required**
- `Environment` abstraction and property sources (Part 35)

**Previous blogs required**
- Part 35 (*Environment Abstraction*)

---

## 2. What You Will Learn

- How `@Value` injects external properties into fields, constructor parameters, and setters
- Property placeholder syntax: `${property.key}`
- Fallback default value syntax: `${property.key:defaultValue}`
- Difference between Property Placeholders `${...}` and SpEL Expressions `#{...}`
- Role of `PropertySourcesPlaceholderConfigurer` vs Spring's built-in embedded value resolver

---

## 3. Syntax Reference

| Expression Syntax | Type | Example | Description |
|---|---|---|---|
| `${server.port}` | Property Placeholder | `@Value("${server.port}")` | Looks up `server.port` in `Environment`. |
| `${server.port:8080}` | Default Fallback | `@Value("${server.port:8080}")` | Uses `8080` if `server.port` is missing. |
| `#{systemProperties['os.name']}` | SpEL Expression | `@Value("#{systemProperties['os.name']}")` | Evaluates Spring Expression Language. |

---

## 4. Complete Working Example

**File:** `src/main/resources/blog36/config.properties`

```properties
app.timeout=5000
app.feature.enabled=true
```

**File:** `src/main/java/com/example/springcore/blog36/domain/ClientConfig.java`

```java
package com.example.springcore.blog36.domain;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ClientConfig {

    private final int timeout;
    private final boolean featureEnabled;
    private final String maxConnections;

    public ClientConfig(
            @Value("${app.timeout:1000}") int timeout,
            @Value("${app.feature.enabled:false}") boolean featureEnabled,
            @Value("${app.max.connections:100}") String maxConnections) { // Uses default '100'
        this.timeout = timeout;
        this.featureEnabled = featureEnabled;
        this.maxConnections = maxConnections;
    }

    public void printConfig() {
        System.out.println("[ClientConfig] Timeout         : " + timeout + "ms");
        System.out.println("[ClientConfig] Feature Enabled : " + featureEnabled);
        System.out.println("[ClientConfig] Max Connections : " + maxConnections + " (Default Used)");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog36/ValueDemoApp.java`

```java
package com.example.springcore.blog36;

import com.example.springcore.blog36.domain.ClientConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

public class ValueDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog36.domain")
    @PropertySource("classpath:blog36/config.properties")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 36 - @Value Property Resolution Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            ClientConfig config = context.getBean(ClientConfig.class);
            config.printConfig();
        }
    }
}
```

Expected Output:

```text
=== Blog 36 - @Value Property Resolution Demo ===
[ClientConfig] Timeout         : 5000ms
[ClientConfig] Feature Enabled : true
[ClientConfig] Max Connections : 100 (Default Used)
```

---

## 5. Key Takeaways

1. Use `@Value("${property.key:default}")` for clean, default-backed property injection.
2. Property placeholders `${...}` resolve against the `Environment`.
3. Automatic type conversion (e.g. `String` to `int` or `boolean`) is performed automatically during property injection.

---

## 6. Next Blog

**Part 37: Type Conversion**
We explore Spring's `ConversionService`, `Converter<S,T>`, and `Formatter<T>` SPIs.

Where you are: **Blog 36 of 49** (Phase 4b: Environment and Infrastructure Abstractions).
