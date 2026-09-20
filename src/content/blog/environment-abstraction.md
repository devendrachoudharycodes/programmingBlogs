---
title: "Environment Abstraction: Profiles, Properties, and PropertySources"
description: "Part 35 of the Spring Core series. Deep dive into Spring's Environment interface, active profiles (@Profile), property resolution, and PropertySources precedence."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 35

tags:
  - java
  - spring
  - spring-core
  - environment
  - profiles

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/environment-abstraction/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 35 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 45 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Environment interface, PropertySources, PropertySourcesPropertyResolver, Profiles (@Profile), property precedence hierarchy |
| **Concepts unlocked** | Part 36 (@Value and Property Resolution), Part 37 (Type Conversion) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` and conditional bean registration (Parts 6 & 34)

**Previous blogs required**
- Part 6 (*BeanFactory vs ApplicationContext*)
- Part 34 (*Conditional Bean Registration*)

---

## 2. What You Will Learn

- The two key pillars of Spring's `Environment` abstraction: **Profiles** and **Properties**
- How `@Profile("dev")` and `@Profile("prod")` activate profile-specific beans
- How `PropertySources` manages property precedence from system properties, environment variables, and `.properties` files
- How to manipulate property sources programmatically using `MutablePropertySources`

---

## 3. Property Sources Precedence Hierarchy

Spring evaluates property sources in order of priority (first match wins):

```text
1. ServletConfig / ServletContext parameters (Web apps)
2. JVM System Properties (-Ddb.url=...)
3. OS Environment Variables (export DB_URL=...)
4. Custom PropertySources (@PropertySource("classpath:app.properties"))
```

---

## 4. Complete Working Example

**File:** `src/main/resources/blog35/app-dev.properties`

```properties
app.name=Spring Core Learning (DEV)
db.url=jdbc:h2:mem:devdb
```

**File:** `src/main/java/com/example/springcore/blog35/domain/DatabaseGateway.java`

```java
package com.example.springcore.blog35.domain;

public interface DatabaseGateway {
    void connect();
}
```

**File:** `src/main/java/com/example/springcore/blog35/domain/H2DevGateway.java`

```java
package com.example.springcore.blog35.domain;

import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Component;

@Component
@Profile("dev") // Active only when 'dev' profile is enabled
public class H2DevGateway implements DatabaseGateway {
    @Override
    public void connect() {
        System.out.println("[H2DevGateway (@Profile('dev'))] Connected to in-memory H2 database!");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog35/EnvironmentDemoApp.java`

```java
package com.example.springcore.blog35;

import com.example.springcore.blog35.domain.DatabaseGateway;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;

public class EnvironmentDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog35.domain")
    @PropertySource("classpath:blog35/app-dev.properties")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 35 - Environment & Profiles Demo ===");

        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {

            // 1. Activate 'dev' profile before refresh
            context.getEnvironment().setActiveProfiles("dev");

            // 2. Register configuration and refresh
            context.register(Config.class);
            context.refresh();

            // 3. Read property from Environment
            Environment env = context.getEnvironment();
            System.out.println("App Name: " + env.getProperty("app.name"));
            System.out.println("DB URL  : " + env.getProperty("db.url"));

            // 4. Test Profile-specific bean
            DatabaseGateway gateway = context.getBean(DatabaseGateway.class);
            gateway.connect();
        }
    }
}
```

Expected Output:

```text
=== Blog 35 - Environment & Profiles Demo ===
App Name: Spring Core Learning (DEV)
DB URL  : jdbc:h2:mem:devdb
[H2DevGateway (@Profile('dev'))] Connected to in-memory H2 database!
```

---

## 5. Key Takeaways

1. `Environment` abstracts profiles and properties into a unified API.
2. Profiles must be set on `Environment` **before** calling `context.refresh()`.
3. `@PropertySource` adds external property files to the environment's `PropertySources` list.

---

## 6. Next Blog

**Part 36: `@Value` and Property Resolution**
We explore `@Value` placeholder injection, `${...}` resolution, and default values.

Where you are: **Blog 35 of 49** (Phase 4b: Environment and Infrastructure Abstractions).
