# Validation

## Input Validation with Bean Validation

---

# Why Validate Input?

<v-clicks>

- **Security** - Prevent injection attacks
- **Data Integrity** - Ensure consistent data
- **User Experience** - Clear error feedback
- **Business Rules** - Enforce constraints

</v-clicks>

---

# Bean Validation (JSR-380)

Standard validation API for Java:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Includes Hibernate Validator as implementation.

---

# Basic Validation Annotations

| Annotation | Purpose |
|------------|---------|
| `@NotNull` | Value must not be null |
| `@NotEmpty` | String/Collection not null or empty |
| `@NotBlank` | String not null, not empty, not whitespace |
| `@Size` | Size/length within bounds |
| `@Min` / `@Max` | Numeric bounds |
| `@Email` | Valid email format |

---

# DTO Validation

```java
public class UserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 150, message = "Invalid age")
    private Integer age;

    // Getters and setters
}
```

---

# @Valid in Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<User> createUser(
            @Valid @RequestBody UserRequest request) {
        User user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @PutMapping("/{id}")
    public User updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserRequest request) {
        return userService.update(id, request);
    }
}
```

---

# Validation Error Response

Without custom handling:
```json
{
    "timestamp": "2024-01-15T10:30:00.000+00:00",
    "status": 400,
    "error": "Bad Request",
    "errors": [...]
}
```

<v-click>

Custom handling provides better response format.

</v-click>

---

# String Validation

```java
@NotBlank
private String name;  // Required, non-empty

@Size(min = 5, max = 50)
private String title;  // Length constraint

@Pattern(regexp = "^[A-Z]{2}\\d{6}$",
         message = "Invalid ID format")
private String passportNumber;  // Regex pattern

@Email(regexp = ".*@company\\.com$",
       message = "Must be company email")
private String workEmail;  // Custom email pattern
```

---

# Numeric Validation

```java
@NotNull
@Min(0)
private Integer quantity;  // Non-negative

@DecimalMin(value = "0.01", message = "Price must be positive")
@DecimalMax(value = "999999.99")
private BigDecimal price;

@Positive
private Integer positiveNumber;  // > 0

@PositiveOrZero
private Integer nonNegative;  // >= 0

@Negative
private Integer negativeNumber;  // < 0
```

---

# Date/Time Validation

```java
@NotNull
@Past(message = "Birth date must be in the past")
private LocalDate birthDate;

@Future(message = "Event date must be in the future")
private LocalDateTime eventDate;

@PastOrPresent
private LocalDate registrationDate;

@FutureOrPresent
private LocalDate startDate;
```

---

# Collection Validation

```java
@NotEmpty(message = "At least one item required")
@Size(max = 10, message = "Maximum 10 items")
private List<@NotBlank String> tags;

@NotNull
@Size(min = 1, max = 5)
private List<@Valid OrderItem> items;  // Nested validation
```

---

# Nested Object Validation

```java
public class OrderRequest {

    @NotNull
    @Valid  // Validate nested object
    private AddressDto shippingAddress;

    @NotEmpty
    private List<@Valid OrderItemDto> items;
}

public class AddressDto {

    @NotBlank
    private String street;

    @NotBlank
    private String city;

    @Pattern(regexp = "\\d{5}", message = "Invalid zip code")
    private String zipCode;
}
```

---

# Custom Validation Annotation

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
public @interface ValidPhoneNumber {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

# Custom Validator

```java
public class PhoneNumberValidator
        implements ConstraintValidator<ValidPhoneNumber, String> {

    private static final Pattern PHONE_PATTERN =
        Pattern.compile("^\\+?[1-9]\\d{9,14}$");

    @Override
    public void initialize(ValidPhoneNumber annotation) {
        // Initialization logic if needed
    }

    @Override
    public boolean isValid(String value,
                          ConstraintValidatorContext context) {
        if (value == null) {
            return true;  // Use @NotNull for null check
        }
        return PHONE_PATTERN.matcher(value).matches();
    }
}
```

---

# Using Custom Annotation

```java
public class ContactRequest {

    @NotBlank
    private String name;

    @ValidPhoneNumber
    private String phoneNumber;
}
```

---

# Class-Level Validation

Validate multiple fields together:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordMatchValidator.class)
public @interface PasswordMatch {
    String message() default "Passwords do not match";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

# Class-Level Validator

```java
public class PasswordMatchValidator
        implements ConstraintValidator<PasswordMatch, PasswordRequest> {

    @Override
    public boolean isValid(PasswordRequest request,
                          ConstraintValidatorContext context) {
        if (request.getPassword() == null) {
            return true;
        }
        return request.getPassword()
                      .equals(request.getConfirmPassword());
    }
}
```

```java
@PasswordMatch
public class PasswordRequest {
    private String password;
    private String confirmPassword;
}
```

---

# Validation Groups

Different validation for different scenarios:

```java
public interface OnCreate {}
public interface OnUpdate {}

public class UserRequest {

    @Null(groups = OnCreate.class)
    @NotNull(groups = OnUpdate.class)
    private Long id;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String name;

    @NotBlank(groups = OnCreate.class)
    private String password;  // Required only on create
}
```

---

# Using Validation Groups

```java
@PostMapping
public User createUser(
        @Validated(OnCreate.class) @RequestBody UserRequest request) {
    return userService.create(request);
}

@PutMapping("/{id}")
public User updateUser(
        @PathVariable Long id,
        @Validated(OnUpdate.class) @RequestBody UserRequest request) {
    return userService.update(id, request);
}
```

Note: Use `@Validated` instead of `@Valid` for groups.

---

# Path Variable Validation

```java
@RestController
@RequestMapping("/api/users")
@Validated  // Enable validation on class
public class UserController {

    @GetMapping("/{id}")
    public User getUser(
            @PathVariable @Min(1) Long id) {
        return userService.findById(id);
    }

    @GetMapping("/search")
    public List<User> search(
            @RequestParam @Size(min = 2) String query) {
        return userService.search(query);
    }
}
```

---

# Programmatic Validation

Validate manually in service:

```java
@Service
public class UserService {

    private final Validator validator;

    public UserService(Validator validator) {
        this.validator = validator;
    }

    public void processUser(UserRequest request) {
        Set<ConstraintViolation<UserRequest>> violations =
            validator.validate(request);

        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
        // Process valid request
    }
}
```

---

# Handling Validation Errors

```java
@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, Object> handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        return Map.of(
            "status", 400,
            "message", "Validation failed",
            "errors", errors
        );
    }
}
```

---

# Validation Best Practices

<v-clicks>

- Validate at API boundaries (controllers)
- Use DTOs with validation, not entities
- Provide clear, user-friendly messages
- Use custom validators for complex rules
- Combine with exception handling
- Don't rely only on client-side validation

</v-clicks>

---

# Summary

<v-clicks>

- `@Valid` triggers validation in controllers
- Built-in annotations: `@NotNull`, `@Size`, `@Email`, etc.
- Nested objects need `@Valid` too
- Custom validators for complex rules
- Validation groups for different scenarios
- Handle `MethodArgumentNotValidException` globally

</v-clicks>
