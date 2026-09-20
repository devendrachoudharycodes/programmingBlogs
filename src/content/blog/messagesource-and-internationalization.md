---
title: "MessageSource and Internationalization (i18n): Multi-Language Message Resolution"
description: "Part 41 of the Spring Core series. How Spring's MessageSource resolves parameterized, localized messages across multiple locales."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 41

tags:
  - java
  - spring
  - spring-core
  - messagesource
  - i18n

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/messagesource-and-internationalization/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 41 of 49 |
| **Difficulty** | 🟢 Beginner |
| **Reading / Coding / Total** | 25 min / 35 min / ~60 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | MessageSource interface, ResourceBundleMessageSource, locale message resolution, parameterized messages |
| **Concepts unlocked** | Part 42 (ApplicationContext Architecture) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` container basics (Parts 3 & 6)

**Previous blogs required**
- Part 6 (*BeanFactory vs ApplicationContext*)

---

## 2. What You Will Learn

- The `MessageSource` interface API for resolving internationalized messages
- Configuring `ResourceBundleMessageSource` for property bundle files
- Resolving parameterized messages with `Object[]` arguments
- Resolving messages by `Locale` (e.g., `Locale.ENGLISH`, `Locale.GERMAN`)

---

## 3. `MessageSource` API

```java
public interface MessageSource {
    String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
    String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
}
```

---

## 4. Complete Working Example

**File:** `src/main/resources/blog41/messages.properties` (Default English)

```properties
welcome.message=Hello {0}, welcome to Spring Core!
order.confirmed=Order {0} successfully placed for ${1}.
```

**File:** `src/main/resources/blog41/messages_de.properties` (German)

```properties
welcome.message=Hallo {0}, willkommen bei Spring Core!
order.confirmed=Bestellung {0} erfolgreich für ${1} platziert.
```

**File:** `src/main/java/com/example/springcore/blog41/I18nDemoApp.java`

```java
package com.example.springcore.blog41;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ResourceBundleMessageSource;

import java.util.Locale;

public class I18nDemoApp {

    @Configuration
    static class Config {

        @Bean(name = "messageSource") // MUST be named 'messageSource'!
        public ResourceBundleMessageSource messageSource() {
            ResourceBundleMessageSource source = new ResourceBundleMessageSource();
            source.setBasename("blog41/messages");
            source.setDefaultEncoding("UTF-8");
            return source;
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Blog 41 - MessageSource i18n Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            // 1. Resolve English Messages
            String welcomeEn = context.getMessage("welcome.message", new Object[]{"Devendra"}, Locale.ENGLISH);
            String orderEn = context.getMessage("order.confirmed", new Object[]{"ORD-101", "99.99"}, Locale.ENGLISH);

            System.out.println("[EN] " + welcomeEn);
            System.out.println("[EN] " + orderEn);

            // 2. Resolve German Messages
            String welcomeDe = context.getMessage("welcome.message", new Object[]{"Devendra"}, Locale.GERMAN);
            String orderDe = context.getMessage("order.confirmed", new Object[]{"ORD-101", "99.99"}, Locale.GERMAN);

            System.out.println("[DE] " + welcomeDe);
            System.out.println("[DE] " + orderDe);
        }
    }
}
```

Expected Output:

```text
=== Blog 41 - MessageSource i18n Demo ===
[EN] Hello Devendra, welcome to Spring Core!
[EN] Order ORD-101 successfully placed for $99.99.
[DE] Hallo Devendra, willkommen bei Spring Core!
[DE] Bestellung ORD-101 erfolgreich für $99.99 platziert.
```

---

## 5. Key Takeaways

1. `ApplicationContext` implements `MessageSource` to resolve localized messages.
2. The `ResourceBundleMessageSource` bean **MUST** be named `messageSource`.
3. Parameterized messages use `{0}`, `{1}` placeholders matching passed argument arrays.

---

## 6. Next Blog

**Part 42: `ApplicationContext` Internal Architecture**
We enter Phase 5, examining `AbstractApplicationContext` and internal framework delegation.

Where you are: **Blog 41 of 49** (Phase 4b: Environment and Infrastructure Abstractions - Complete!).
