# Configuration

## Properties, Profiles, and Environment

---

# Configuration Sources

Spring Boot loads configuration from multiple sources (in priority order):

<v-clicks>

1. Command line arguments
2. Environment variables
3. `application.properties` / `application.yml`
4. Default properties

</v-clicks>

---

# application.properties

```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Application
app.name=My Application
app.version=1.0.0

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.file.name=app.log
```

---

# application.yml

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

app:
  name: My Application
  version: 1.0.0

logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: app.log
```

---

# @Value Annotation

Inject property values:

```java
@Service
public class MyService {

    @Value("${app.name}")
    private String appName;

    @Value("${app.timeout:5000}")  // With default
    private int timeout;

    @Value("${app.features.enabled:false}")
    private boolean featuresEnabled;
}
```

---

# @ConfigurationProperties

Type-safe configuration binding:

```java
@Configuration
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private String version;
    private Security security = new Security();

    // Getters and setters

    public static class Security {
        private String secretKey;
        private int tokenExpiry;
        // Getters and setters
    }
}
```

---

# Using @ConfigurationProperties

```yaml
app:
  name: My Application
  version: 1.0.0
  security:
    secret-key: mySecretKey123
    token-expiry: 3600
```

```java
@Service
public class TokenService {

    private final AppProperties appProperties;

    public TokenService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public String createToken() {
        String secret = appProperties.getSecurity().getSecretKey();
        // ...
    }
}
```

---

# Enable Configuration Properties

```java
@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Or use `@ConfigurationPropertiesScan`:
```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class DemoApplication { }
```

---

# Profiles

Different configurations for different environments:

```
application.properties         # Common
application-dev.properties     # Development
application-prod.properties    # Production
application-test.properties    # Testing
```

---

# Profile-Specific Configuration

```properties
# application-dev.properties
server.port=8080
spring.datasource.url=jdbc:h2:mem:devdb
spring.jpa.show-sql=true
logging.level.com.example=DEBUG
```

```properties
# application-prod.properties
server.port=80
spring.datasource.url=jdbc:postgresql://prod-db:5432/myapp
spring.jpa.show-sql=false
logging.level.com.example=WARN
```

---

# Activating Profiles

```properties
# In application.properties
spring.profiles.active=dev
```

```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Environment variable
export SPRING_PROFILES_ACTIVE=prod
```

---

# Multiple Active Profiles

```properties
spring.profiles.active=dev,metrics
```

Or:
```bash
java -jar app.jar --spring.profiles.active=dev,metrics
```

Profile-specific files are loaded in order.

---

# @Profile Annotation

Conditional bean creation:

```java
@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public DataSource dataSource() {
        return new H2DataSource();
    }
}

@Configuration
@Profile("prod")
public class ProdConfig {

    @Bean
    public DataSource dataSource() {
        return new PostgresDataSource();
    }
}
```

---

# Environment Variables

```properties
# Reference environment variables
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

# With default values
spring.datasource.url=${DATABASE_URL:jdbc:h2:mem:testdb}
```

<v-click>

Setting environment variables:
```bash
export DB_USERNAME=admin
export DB_PASSWORD=secret
java -jar app.jar
```

</v-click>

---

# Externalized Configuration

Priority (highest to lowest):

1. `--property=value` command line
2. `SPRING_APPLICATION_JSON` environment variable
3. OS environment variables
4. Profile-specific properties
5. Application properties
6. `@PropertySource` annotations
7. Default properties

---

# Configuration in Docker

```dockerfile
FROM eclipse-temurin:17-jre
COPY target/app.jar app.jar
ENV SPRING_PROFILES_ACTIVE=prod
ENV DB_PASSWORD=secret
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Or use docker-compose:
```yaml
services:
  app:
    image: myapp
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_PASSWORD=secret
```

---

# Random Values

Spring Boot can generate random values:

```properties
# Random int
app.secret=${random.int}

# Random int in range
app.port=${random.int[8000,8100]}

# Random long
app.bigNumber=${random.long}

# Random UUID
app.id=${random.uuid}

# Random string
app.token=${random.value}
```

---

# Property Validation

Validate configuration properties:

```java
@Configuration
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {

    @NotBlank
    private String name;

    @Min(1)
    @Max(65535)
    private int port;

    @Email
    private String adminEmail;

    // Getters and setters
}
```

---

# @Conditional Annotations

Create beans conditionally:

```java
@Bean
@ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
public CacheManager cacheManager() {
    return new ConcurrentMapCacheManager();
}

@Bean
@ConditionalOnMissingBean
public MyService defaultMyService() {
    return new DefaultMyService();
}

@Bean
@ConditionalOnClass(name = "com.redis.RedisClient")
public CacheManager redisCacheManager() {
    return new RedisCacheManager();
}
```

---

# Common Properties Reference

```properties
# Server
server.port=8080
server.address=0.0.0.0
server.tomcat.max-threads=200

# Database
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
spring.datasource.hikari.maximum-pool-size=10

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# Jackson (JSON)
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=UTC
```

---

# Summary

<v-clicks>

- `application.properties` or `application.yml` for configuration
- `@Value` for simple property injection
- `@ConfigurationProperties` for type-safe binding
- Profiles for environment-specific config
- Environment variables for sensitive data
- `@Profile` for conditional beans
- Configuration priority determines which value wins

</v-clicks>
