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

# Application info
spring.application.name=my-app

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
  application:
    name: my-app

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

# Server port
server.port=8080

# Detailed logging
logging.level.com.example=DEBUG
logging.level.org.springframework.web=DEBUG
```

---

# Prod Profile Example

```properties
# application-prod.properties

# Server port
server.port=80

# Minimal logging
logging.level.com.example=WARN
logging.level.org.springframework.web=WARN
```

---

# Environment Variables

Use placeholders for sensitive data:

```properties
app.api-key=${API_KEY}
app.secret=${APP_SECRET:defaultSecret}
```

<v-click>

Set environment variables:
```bash
export API_KEY=your-api-key
export APP_SECRET=secret123
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
        └── service/
            └── UserServiceTest.java
```

---

# Recommended Project Structure

```
com.example.demo/
├── DemoApplication.java
├── config/
│   └── WebConfig.java
├── controller/
│   └── UserController.java
├── service/
│   └── UserService.java
├── model/
│   ├── User.java
│   ├── UserRequest.java
│   └── UserResponse.java
└── exception/
    └── UserNotFoundException.java
```

---

# The Boilerplate Problem

A simple model class requires so much code:

```java
public class User {
    private Long id;
    private String name;
    private String email;

    public User() {}
    public User(Long id, String name, String email) {
        this.id = id; this.name = name; this.email = email;
    }
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // ... equals(), hashCode(), toString() ...
}
```

😫 50+ lines for 3 fields!

---

# Lombok to the Rescue

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
    private String email;
}
```

<v-click>

✅ Same functionality in 8 lines!

`@Data` generates: getters, setters, `toString()`, `equals()`, `hashCode()`

</v-click>

---

# Common Lombok Annotations

| Annotation | What it generates |
|------------|------------------|
| `@Getter` / `@Setter` | Getters and setters |
| `@NoArgsConstructor` | Empty constructor |
| `@AllArgsConstructor` | Constructor with all fields |
| `@Data` | All of the above + toString, equals, hashCode |
| `@Builder` | Builder pattern |

---

# Lombok @Builder

```java
@Data
@Builder
public class User {
    private Long id;
    private String name;
    private String email;
}
```

```java
User user = User.builder()
    .id(1L)
    .name("John")
    .email("john@example.com")
    .build();
```

---

# Model vs DTO

**Model** - Internal representation:
```java
@Data
public class User {
    private Long id;
    private String name;
    private String email;
    private String password; // Sensitive!
}
```

**DTO** - Data transfer object (API response):
```java
@Data
public class UserResponse {
    private Long id;
    private String name;
    private String email;
    // No password exposed!
}
```

---

# The Mapping Problem

Converting Model → DTO manually is tedious:

```java
public UserResponse toResponse(User user) {
    UserResponse response = new UserResponse();
    response.setId(user.getId());
    response.setName(user.getName());
    response.setEmail(user.getEmail());
    return response;
}
```

<v-click>

😫 Imagine doing this for 20 fields... and for every model!

</v-click>

---

# MapStruct to the Rescue

```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    UserResponse toResponse(User user);

    User toModel(UserRequest request);

    List<UserResponse> toResponseList(List<User> users);
}
```

<v-click>

✅ MapStruct generates the implementation at compile time!

</v-click>

---

# Using MapStruct

```java
@Service
public class UserService {

    private final UserMapper userMapper;

    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public UserResponse getUser(Long id) {
        User user = findById(id);
        return userMapper.toResponse(user);  // Auto-maps fields!
    }
}
```

---

# MapStruct Custom Mapping

When field names don't match:

```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(source = "name", target = "fullName")
    @Mapping(target = "id", ignore = true)
    UserResponse toResponse(User user);
}
```

---

# Layered Architecture

```
┌─────────────────────────────────────────┐
│           Controller Layer              │
│         (REST endpoints, HTTP)          │
├─────────────────────────────────────────┤
│            Service Layer                │
│          (Business logic)               │
├─────────────────────────────────────────┤
│          Repository Layer               │
│       (Data access, storage)            │
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
- Keep models and DTOs separate

</v-clicks>
