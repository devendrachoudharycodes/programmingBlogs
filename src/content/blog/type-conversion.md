---
title: "Type Conversion: ConversionService, Converter, and Formatter SPIs"
description: "Part 37 of the Spring Core series. How Spring performs automatic type conversion using ConversionService, custom Converter<S,T>, and Formatter<T> SPIs."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 37

tags:
  - java
  - spring
  - spring-core
  - type-conversion
  - conversion-service

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/type-conversion/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 37 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | ConversionService API, Converter<S,T> interface, PropertyEditor legacy vs ConversionService, FormattingConversionService |
| **Concepts unlocked** | Part 38 (SpEL Fundamentals), Part 47 (Dependency Resolution Internals) |

## 1. Prerequisites

**Spring concepts required**
- Property injection via `@Value` (Part 36)

**Previous blogs required**
- Part 36 (*@Value and Property Resolution*)

---

## 2. What You Will Learn

- How Spring converts String properties to complex Java types (e.g. `String` to `URL`, `Duration`, or custom Domain Enums)
- The legacy JavaBeans `PropertyEditor` vs modern thread-safe `ConversionService`
- Implementing the `Converter<S, T>` SPI interface for custom type conversions
- Registering custom converters in `ConversionServiceFactoryBean`

---

## 3. `Converter<S, T>` Interface

```java
@FunctionalInterface
public interface Converter<S, T> {
    T convert(S source);
}
```

Implementations must be **thread-safe** and stateless.

---

## 4. Complete Working Example (Custom String ──► OrderId Converter)

**File:** `src/main/java/com/example/springcore/blog37/domain/OrderId.java`

```java
package com.example.springcore.blog37.domain;

public record OrderId(String value) {
    @Override
    public String toString() {
        return "ORDER-ID[" + value + "]";
    }
}
```

**File:** `src/main/java/com/example/springcore/blog37/converter/StringToOrderIdConverter.java`

```java
package com.example.springcore.blog37.converter;

import com.example.springcore.blog37.domain.OrderId;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;

@Component
public class StringToOrderIdConverter implements Converter<String, OrderId> {

    @Override
    public OrderId convert(String source) {
        return new OrderId(source.trim().toUpperCase());
    }
}
```

**File:** `src/main/java/com/example/springcore/blog37/domain/OrderReceiver.java`

```java
package com.example.springcore.blog37.domain;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class OrderReceiver {

    private final OrderId activeOrderId;

    // Custom String to OrderId conversion happens automatically via ConversionService!
    public OrderReceiver(@Value("${app.default.order-id:ord-9999}") OrderId activeOrderId) {
        this.activeOrderId = activeOrderId;
    }

    public void process() {
        System.out.println("[OrderReceiver] Processing active order: " + activeOrderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog37/ConversionDemoApp.java`

```java
package com.example.springcore.blog37;

import com.example.springcore.blog37.converter.StringToOrderIdConverter;
import com.example.springcore.blog37.domain.OrderReceiver;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ConversionServiceFactoryBean;

import java.util.Set;

public class ConversionDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog37")
    static class Config {

        @Bean(name = "conversionService") // Must be named 'conversionService'!
        public ConversionServiceFactoryBean conversionService(StringToOrderIdConverter customConverter) {
            ConversionServiceFactoryBean factory = new ConversionServiceFactoryBean();
            factory.setConverters(Set.of(customConverter));
            return factory;
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Blog 37 - Type Conversion SPI Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            OrderReceiver receiver = context.getBean(OrderReceiver.class);
            receiver.process();
        }
    }
}
```

Expected Output:

```text
=== Blog 37 - Type Conversion SPI Demo ===
[OrderReceiver] Processing active order: ORDER-ID[ORD-9999]
```

---

## 5. Key Takeaways

1. `ConversionService` is Spring's thread-safe, stateless type conversion engine.
2. Implement `Converter<S, T>` to convert String properties or raw inputs to domain objects.
3. Register custom converters in a `ConversionServiceFactoryBean` bean named `conversionService`.

---

## 6. Next Blog

**Part 38: SpEL Fundamentals**
We explore Spring Expression Language syntax, bean references, method calls, and mathematical/logical operators.

Where you are: **Blog 37 of 49** (Phase 4b: Environment and Infrastructure Abstractions).
