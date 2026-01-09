# Testing

## Unit and Integration Tests

---

# Why Test?

<v-clicks>

- **Confidence** - Code works as expected
- **Refactoring** - Change without fear
- **Documentation** - Tests show how code works
- **Regression** - Catch bugs early

</v-clicks>

---

# Testing Pyramid

```
          /\
         /  \
        / E2E\        Few
       /______\
      /        \
     /Integration\    Some
    /______________\
   /                \
  /    Unit Tests    \  Many
 /____________________\
```

---

# Test Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Includes:
- JUnit 5
- Mockito
- AssertJ
- Spring Test

---

# Unit Test Basics

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

class CalculatorTest {

    @Test
    void shouldAddTwoNumbers() {
        Calculator calculator = new Calculator();

        int result = calculator.add(2, 3);

        assertThat(result).isEqualTo(5);
    }
}
```

---

# Test Naming Conventions

```java
// Method name describes behavior
@Test
void shouldReturnEmptyListWhenNoUsersExist() { }

@Test
void shouldThrowExceptionWhenUserNotFound() { }

@Test
void shouldCreateUserWithValidData() { }

// Or use @DisplayName
@Test
@DisplayName("Should return user when valid ID provided")
void getUser_validId_returnsUser() { }
```

---

# Service Unit Test

```java
class UserServiceTest {

    private UserService userService;

    @BeforeEach
    void setUp() {
        userService = new UserService();
    }

    @Test
    void shouldReturnUserWhenFound() {
        // Arrange
        User created = userService.create("John", "john@example.com");

        // Act
        User result = userService.findById(created.getId());

        // Assert
        assertThat(result.getName()).isEqualTo("John");
        assertThat(result.getEmail()).isEqualTo("john@example.com");
    }
}
```

---

# Testing Exceptions

```java
@Test
void shouldThrowExceptionWhenUserNotFound() {
    assertThatThrownBy(() -> userService.findById(99L))
        .isInstanceOf(ResourceNotFoundException.class)
        .hasMessageContaining("User not found");
}
```

Or using JUnit:
```java
@Test
void shouldThrowException() {
    assertThrows(ResourceNotFoundException.class,
        () -> userService.findById(99L));
}
```

---

# @BeforeEach and @AfterEach

```java
class UserServiceTest {

    private UserService userService;

    @BeforeEach
    void setUp() {
        userService = new UserService();
    }

    @AfterEach
    void tearDown() {
        // Cleanup if needed
    }

    @Test
    void testMethod() {
        // Test uses fresh userService
    }
}
```

---

# Parameterized Tests

```java
@ParameterizedTest
@ValueSource(strings = {"", " ", "   "})
void shouldRejectBlankNames(String name) {
    assertThatThrownBy(() -> userService.create(name, "email@test.com"))
        .isInstanceOf(BadRequestException.class);
}

@ParameterizedTest
@CsvSource({
    "john@example.com, true",
    "invalid-email, false",
    "'', false"
})
void shouldValidateEmail(String email, boolean expected) {
    assertThat(validator.isValidEmail(email)).isEqualTo(expected);
}
```

---

# Controller Unit Test with MockMvc

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(user);

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

---

# Testing POST Requests

```java
@Test
void shouldCreateUser() throws Exception {
    UserRequest request = new UserRequest("John", "john@example.com");
    User created = new User(1L, "John", "john@example.com");
    when(userService.create(any())).thenReturn(created);

    mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content("""
                {
                    "name": "John",
                    "email": "john@example.com"
                }
                """))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.id").value(1));
}
```

---

# Testing Validation

```java
@Test
void shouldReturn400ForInvalidInput() throws Exception {
    mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content("""
                {
                    "name": "",
                    "email": "invalid-email"
                }
                """))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.errors.name").exists())
        .andExpect(jsonPath("$.errors.email").exists());
}
```

---

# Integration Tests

Test complete slices of the application:

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldCreateAndRetrieveUser() throws Exception {
        // Create user
        MvcResult result = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"John\",\"email\":\"john@test.com\"}"))
            .andExpect(status().isCreated())
            .andReturn();

        // Extract ID from response
        String response = result.getResponse().getContentAsString();

        // Verify user can be retrieved
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
}
```

---

# Test Profiles

```java
@SpringBootTest
@ActiveProfiles("test")
class UserServiceIntegrationTest {
    // Uses application-test.properties
}
```

```properties
# application-test.properties
logging.level.com.example=DEBUG
server.port=0
```

---

# @TestConfiguration

Provide test-specific beans:

```java
@SpringBootTest
class UserServiceTest {

    @TestConfiguration
    static class TestConfig {
        @Bean
        public EmailService emailService() {
            return new FakeEmailService();
        }
    }

    @Autowired
    private UserService userService;

    @Test
    void shouldSendWelcomeEmail() {
        userService.create("John", "john@test.com");
        // Uses FakeEmailService
    }
}
```

---

# AssertJ Assertions

```java
// Basic
assertThat(user.getName()).isEqualTo("John");
assertThat(user.getName()).startsWith("Jo");
assertThat(user.getAge()).isGreaterThan(18);

// Collections
assertThat(users).hasSize(3);
assertThat(users).contains(user1, user2);
assertThat(users).extracting("name").contains("John", "Jane");

// Exceptions
assertThatThrownBy(() -> service.fail())
    .isInstanceOf(RuntimeException.class)
    .hasMessage("Error");

// Optional
assertThat(optional).isPresent().hasValue(expected);
```

---

# Test Coverage

Run with coverage in IDE or:

```bash
# Maven
mvn test jacoco:report

# Gradle
gradle test jacocoTestReport
```

<v-click>

Aim for meaningful coverage, not 100%.

Focus on:
- Business logic
- Edge cases
- Error handling

</v-click>

---

# Testing Best Practices

<v-clicks>

- **Arrange-Act-Assert** pattern
- **One assertion per test** (when possible)
- **Test behavior, not implementation**
- **Use meaningful test names**
- **Keep tests independent**
- **Fast tests for quick feedback**
- **Don't test framework code**

</v-clicks>

---

# Summary

<v-clicks>

- JUnit 5 for test framework
- Mockito for mocking dependencies
- `@WebMvcTest` for controller tests
- `@SpringBootTest` for integration tests
- AssertJ for fluent assertions
- Test profiles for isolated configuration

</v-clicks>
