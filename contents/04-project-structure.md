# Project Structure

## Organizing a Spring Boot Application

---

# Standard Project Layout

```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/demo/
│   │   │       ├── DemoApplication.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       └── config/
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/
│   │       └── templates/
│   └── test/
├── pom.xml
└── README.md
```

---

# Package by Layer

Traditional approach - group by technical function:

```
com.example.demo/
├── controller/
│   ├── UserController.java
│   └── ProductController.java
├── service/
│   ├── UserService.java
│   └── ProductService.java
├── repository/
│   ├── UserRepository.java
│   └── ProductRepository.java
└── model/
    ├── User.java
    └── Product.java
```

---

# Package by Feature

Modern approach - group by business domain:

```
com.example.demo/
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   └── User.java
├── product/
│   ├── ProductController.java
│   ├── ProductService.java
│   ├── ProductRepository.java
│   └── Product.java
└── common/
    └── ...
```

---

# Layer vs Feature Comparison

| Package by Layer | Package by Feature |
|-----------------|-------------------|
| Easy to navigate by type | Easy to navigate by domain |
| Good for small projects | Scales better |
| Technical grouping | Business grouping |
| Cross-cutting changes | Isolated changes |

---

# The Main Application Class

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

# @SpringBootApplication Explained

```java
@SpringBootApplication
// Equivalent to:
@Configuration           // Allows @Bean definitions
@EnableAutoConfiguration // Auto-configure based on classpath
@ComponentScan           // Scan for components in package
```

---

# Resources Directory

```
resources/
├── application.properties    # Main configuration
├── application.yml           # Alternative YAML format
├── application-dev.properties  # Dev profile
├── application-prod.properties # Prod profile
├── static/                   # Static files (CSS, JS)
│   ├── css/
│   └── js/
└── templates/                # Template files (Thymeleaf)
```

---

# application.properties

```properties
# Server configuration
server.port=8080

# Database configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver

# JPA configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
```

---

# application.yml Alternative

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

logging:
  level:
    root: INFO
    com.example: DEBUG
```

---

# Properties vs YAML

| Properties | YAML |
|------------|------|
| Flat structure | Hierarchical |
| Simple | More readable |
| No indentation issues | Indentation matters |
| Spring Boot default | Common alternative |

---

# Profile-Specific Configuration

```
application.properties          # Common settings
application-dev.properties      # Development
application-prod.properties     # Production
application-test.properties     # Testing
```

<v-click>

Activate profile:
```properties
# application.properties
spring.profiles.active=dev
```

Or via command line:
```bash
java -jar app.jar --spring.profiles.active=prod
```

</v-click>

---

# Dev Profile Example

```properties
# application-dev.properties

# Use H2 in-memory database
spring.datasource.url=jdbc:h2:mem:devdb
spring.h2.console.enabled=true

# Show SQL queries
spring.jpa.show-sql=true

# Detailed logging
logging.level.com.example=DEBUG
```

---

# Prod Profile Example

```properties
# application-prod.properties

# Use PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/proddb
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}

# Validate schema only
spring.jpa.hibernate.ddl-auto=validate

# Less logging
logging.level.com.example=WARN
```

---

# Environment Variables

Use placeholders for sensitive data:

```properties
spring.datasource.username=${DB_USER:defaultUser}
spring.datasource.password=${DB_PASSWORD}
```

<v-click>

Set environment variables:
```bash
export DB_USER=admin
export DB_PASSWORD=secret123
java -jar app.jar
```

</v-click>

---

# The pom.xml Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Dependencies here -->
    </dependencies>
</project>
```

---

# Spring Boot Starters

Pre-packaged dependency bundles:

| Starter | Includes |
|---------|----------|
| spring-boot-starter-web | Spring MVC, Tomcat, Jackson |
| spring-boot-starter-data-jpa | JPA, Hibernate, HikariCP |
| spring-boot-starter-security | Spring Security |
| spring-boot-starter-test | JUnit, Mockito, AssertJ |

---

# Test Directory Structure

```
test/
└── java/
    └── com/example/demo/
        ├── DemoApplicationTests.java
        ├── controller/
        │   └── UserControllerTest.java
        ├── service/
        │   └── UserServiceTest.java
        └── repository/
            └── UserRepositoryTest.java
```

---

# Recommended Project Structure

```
com.example.demo/
├── DemoApplication.java
├── config/
│   ├── WebConfig.java
│   └── SecurityConfig.java
├── controller/
│   └── UserController.java
├── service/
│   ├── UserService.java
│   └── impl/
│       └── UserServiceImpl.java
├── repository/
│   └── UserRepository.java
├── model/
│   ├── entity/
│   │   └── User.java
│   └── dto/
│       ├── UserRequest.java
│       └── UserResponse.java
└── exception/
    └── UserNotFoundException.java
```

---

# DTO vs Entity

**Entity** - Maps to database table:
```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    private String password; // Sensitive!
}
```

**DTO** - Data transfer object:
```java
public class UserResponse {
    private Long id;
    private String name;
    // No password exposed!
}
```

---

# Layered Architecture

```
┌─────────────────────────────────────────┐
│           Controller Layer              │
│      (REST endpoints, validation)       │
├─────────────────────────────────────────┤
│            Service Layer                │
│       (Business logic, transactions)    │
├─────────────────────────────────────────┤
│          Repository Layer               │
│         (Data access, queries)          │
├─────────────────────────────────────────┤
│              Database                   │
└─────────────────────────────────────────┘
```

---

# Summary

<v-clicks>

- Standard Maven/Gradle project structure
- Package by layer (traditional) or feature (modern)
- `@SpringBootApplication` combines three annotations
- `application.properties` or `application.yml` for configuration
- Profiles for environment-specific settings
- Use environment variables for sensitive data
- Keep entities and DTOs separate

</v-clicks>
