# Spring Core Concepts

## IoC, DI, and Beans

---

# Inversion of Control (IoC)

**Traditional approach:** Objects create their dependencies

```java
public class OrderService {
    private PaymentService paymentService = new PaymentService();
}
```

<v-click>

**IoC approach:** Container manages object creation

```java
public class OrderService {
    private PaymentService paymentService; // Injected by Spring
}
```

</v-click>

---

# IoC Principle

```
Traditional:                    IoC:
+-------------+                +-------------+
| OrderService|                |  Container  |
|  creates    |                |   creates   |
|      |      |                |      |      |
|      v      |                |      v      |
|PaymentService|               |   Objects   |
+-------------+                +-------------+
```

> "Don't call us, we'll call you" - Hollywood Principle

---

# Dependency Injection (DI)

DI is a **pattern** to implement IoC

<v-clicks>

Three types of injection:
1. **Constructor Injection** (Recommended)
2. **Setter Injection**
3. **Field Injection** (Not recommended)

</v-clicks>

---

# Constructor Injection

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    // Spring automatically injects dependencies
    public OrderService(PaymentService paymentService,
                       InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

<v-click>

**Benefits:**
- Dependencies are clearly visible
- Immutable (final fields)
- Easy to test
- Required dependencies enforced

</v-click>

---

# Setter Injection

```java
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

<v-click>

**Use when:**
- Dependencies are optional
- Circular dependencies exist (try to avoid)

</v-click>

---

# Field Injection (Avoid)

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService; // Not recommended!
}
```

<v-click>

**Why avoid:**
- Hidden dependencies
- Hard to test
- Cannot make fields final
- Tight coupling with Spring

</v-click>

---

# What is a Bean?

A **Bean** is an object managed by the Spring IoC container

```java
@Component
public class MyService {
    // This class becomes a Spring Bean
}
```

<v-click>

Spring manages:
- Creation
- Configuration
- Lifecycle
- Destruction

</v-click>

---

# Bean Stereotypes

| Annotation | Purpose |
|------------|---------|
| `@Component` | Generic component |
| `@Service` | Business logic |
| `@Repository` | Data access |
| `@Controller` | Web controller |
| `@RestController` | REST API controller |

---

# @Component

```java
@Component
public class EmailValidator {

    public boolean isValid(String email) {
        return email != null && email.contains("@");
    }
}
```

Generic stereotype for any Spring-managed component.

---

# @Service

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(String name, String email) {
        User user = new User(name, email);
        return userRepository.save(user);
    }
}
```

Indicates a service containing business logic.

---

# @Repository

```java
@Repository
public class UserRepository {

    public User save(User user) {
        // Save to database
        return user;
    }

    public Optional<User> findById(Long id) {
        // Find from database
        return Optional.empty();
    }
}
```

Indicates data access layer, enables exception translation.

---

# @Controller vs @RestController

```java
@Controller
public class WebController {
    @GetMapping("/page")
    public String showPage() {
        return "page"; // Returns view name
    }
}
```

```java
@RestController  // = @Controller + @ResponseBody
public class ApiController {
    @GetMapping("/api/data")
    public Data getData() {
        return new Data(); // Returns JSON
    }
}
```

---

# Bean Scope

| Scope | Description |
|-------|-------------|
| `singleton` | One instance per container (default) |
| `prototype` | New instance each time |
| `request` | One per HTTP request |
| `session` | One per HTTP session |

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // New instance created each time
}
```

---

# Singleton Scope (Default)

```java
@Service
public class CounterService {
    private int count = 0;

    public int increment() {
        return ++count;
    }
}
```

<v-click>

**Warning:** Same instance shared across all users!

All requests share the same `count` variable.

</v-click>

---

# Application Context

The **ApplicationContext** is the Spring IoC container

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        ApplicationContext ctx =
            SpringApplication.run(DemoApplication.class, args);

        // Get bean from container
        UserService userService = ctx.getBean(UserService.class);
    }
}
```

---

# Component Scanning

Spring automatically discovers beans in packages

```java
@SpringBootApplication  // Enables component scanning
public class DemoApplication {
    // Scans com.example.demo and sub-packages
}
```

<v-click>

Custom scanning:
```java
@ComponentScan(basePackages = {"com.example.services",
                               "com.example.repositories"})
```

</v-click>

---

# @Configuration and @Bean

Manual bean definition:

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }
}
```

---

# When to Use @Bean

<v-clicks>

- Third-party classes (can't add @Component)
- Complex initialization logic
- Conditional bean creation
- Multiple beans of same type

</v-clicks>

---

# @Qualifier

When multiple beans of same type exist:

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @Qualifier("primary")
    public DataSource primaryDataSource() { /* ... */ }

    @Bean
    @Qualifier("secondary")
    public DataSource secondaryDataSource() { /* ... */ }
}
```

```java
@Service
public class UserService {
    public UserService(@Qualifier("primary") DataSource dataSource) {
        // Uses primary data source
    }
}
```

---

# @Primary

Mark a bean as the default choice:

```java
@Bean
@Primary
public DataSource primaryDataSource() {
    return new HikariDataSource();
}

@Bean
public DataSource backupDataSource() {
    return new HikariDataSource();
}
```

When injecting `DataSource`, the `@Primary` bean is used by default.

---

# Bean Lifecycle

```
Instantiate → Populate Properties → BeanNameAware →
BeanFactoryAware → Pre-initialization → InitializingBean →
Custom init → Post-initialization → Ready →
DisposableBean → Custom destroy → Destroyed
```

---

# Lifecycle Callbacks

```java
@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Bean initialized");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Bean destroyed");
    }
}
```

---

# Summary

<v-clicks>

- **IoC** - Container manages object creation
- **DI** - Dependencies are injected, not created
- **Beans** - Objects managed by Spring
- **Stereotypes** - @Component, @Service, @Repository, @Controller
- **Scope** - Singleton (default), Prototype, Request, Session
- **Configuration** - @Configuration + @Bean for manual setup

</v-clicks>
