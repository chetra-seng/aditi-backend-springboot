# Exception Handling

## Graceful Error Management

---

# Why Proper Exception Handling?

<v-clicks>

- **User Experience** - Clear, helpful error messages
- **Security** - Don't expose internal details
- **Debugging** - Proper logging for troubleshooting
- **API Contract** - Consistent error responses

</v-clicks>

---

# Default Spring Boot Error Response

Without custom handling:

```json
{
    "timestamp": "2024-01-15T10:30:00.000+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/api/users/1"
}
```

<v-click>

**Problems:**
- Generic message
- May expose sensitive info
- Not API-friendly

</v-click>

---

# Custom Exception Classes

```java
public class ResourceNotFoundException extends RuntimeException {

    private final String resourceName;
    private final String fieldName;
    private final Object fieldValue;

    public ResourceNotFoundException(String resourceName,
                                     String fieldName,
                                     Object fieldValue) {
        super(String.format("%s not found with %s: '%s'",
              resourceName, fieldName, fieldValue));
        this.resourceName = resourceName;
        this.fieldName = fieldName;
        this.fieldValue = fieldValue;
    }

    // Getters
}
```

---

# More Exception Classes

```java
public class BadRequestException extends RuntimeException {
    public BadRequestException(String message) {
        super(message);
    }
}

public class ConflictException extends RuntimeException {
    public ConflictException(String message) {
        super(message);
    }
}

public class UnauthorizedException extends RuntimeException {
    public UnauthorizedException(String message) {
        super(message);
    }
}
```

---

# Using Custom Exceptions

```java
@Service
public class UserService {

    private final Map<Long, User> users = new HashMap<>();

    public User findById(Long id) {
        User user = users.get(id);
        if (user == null) {
            throw new ResourceNotFoundException("User", "id", id);
        }
        return user;
    }

    public User createUser(UserRequest request) {
        boolean emailExists = users.values().stream()
            .anyMatch(u -> u.getEmail().equals(request.getEmail()));
        if (emailExists) {
            throw new ConflictException(
                "Email already registered: " + request.getEmail());
        }
        // Create user...
    }
}
```

---

# Error Response DTO

```java
public class ErrorResponse {

    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;

    // Constructor, getters, setters

    public static ErrorResponse of(HttpStatus status,
                                   String message,
                                   String path) {
        ErrorResponse response = new ErrorResponse();
        response.setTimestamp(LocalDateTime.now());
        response.setStatus(status.value());
        response.setError(status.getReasonPhrase());
        response.setMessage(message);
        response.setPath(path);
        return response;
    }
}
```

---

# @ExceptionHandler

Handle exceptions at controller level:

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex,
            HttpServletRequest request) {
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.NOT_FOUND,
            ex.getMessage(),
            request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

---

# @ControllerAdvice

Global exception handling:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex,
            HttpServletRequest request) {
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.NOT_FOUND,
            ex.getMessage(),
            request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

---

# Complete Global Handler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex,
                                        HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ErrorResponse.of(HttpStatus.NOT_FOUND,
                               ex.getMessage(),
                               request.getRequestURI());
    }

    @ExceptionHandler(BadRequestException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleBadRequest(BadRequestException ex,
                                          HttpServletRequest request) {
        log.warn("Bad request: {}", ex.getMessage());
        return ErrorResponse.of(HttpStatus.BAD_REQUEST,
                               ex.getMessage(),
                               request.getRequestURI());
    }
}
```

---

# Handler Continued

```java
    @ExceptionHandler(ConflictException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleConflict(ConflictException ex,
                                        HttpServletRequest request) {
        return ErrorResponse.of(HttpStatus.CONFLICT,
                               ex.getMessage(),
                               request.getRequestURI());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex,
                                       HttpServletRequest request) {
        log.error("Unexpected error", ex);
        return ErrorResponse.of(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "An unexpected error occurred",  // Don't expose details!
            request.getRequestURI()
        );
    }
```

---

# Validation Exception Handling

Handle `@Valid` validation errors:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public ErrorResponse handleValidation(
        MethodArgumentNotValidException ex,
        HttpServletRequest request) {

    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors().forEach(error ->
        errors.put(error.getField(), error.getDefaultMessage())
    );

    ErrorResponse response = ErrorResponse.of(
        HttpStatus.BAD_REQUEST,
        "Validation failed",
        request.getRequestURI()
    );
    response.setErrors(errors);
    return response;
}
```

---

# Enhanced Error Response

```java
public class ErrorResponse {

    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> errors;  // Field errors
    private String traceId;               // For debugging

    // ...
}
```

---

# Error Response Example

```json
{
    "timestamp": "2024-01-15T10:30:00",
    "status": 400,
    "error": "Bad Request",
    "message": "Validation failed",
    "path": "/api/users",
    "errors": {
        "email": "must be a valid email",
        "name": "must not be blank"
    },
    "traceId": "abc123"
}
```

---

# @ResponseStatus Annotation

Simple exception-to-status mapping:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

<v-click>

Throwing this exception automatically returns 404.

</v-click>

---

# ResponseStatusException

Quick inline exception handling:

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    if (user == null) {
        throw new ResponseStatusException(
            HttpStatus.NOT_FOUND,
            "User not found with id: " + id
        );
    }
    return user;
}
```

---

# Exception Handler Priority

```
1. @ExceptionHandler in Controller
        ↓ (if not found)
2. @ExceptionHandler in @ControllerAdvice
        ↓ (if not found)
3. Default Spring Boot error handling
```

---

# Problem Details (RFC 7807)

Spring 6+ supports Problem Details:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND,
            ex.getMessage()
        );
        problem.setTitle("Resource Not Found");
        problem.setProperty("resourceName", ex.getResourceName());
        return problem;
    }
}
```

---

# Problem Details Response

```json
{
    "type": "about:blank",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "User not found with id: 123",
    "instance": "/api/users/123",
    "resourceName": "User"
}
```

Enable with:
```properties
spring.mvc.problemdetails.enabled=true
```

---

# Best Practices

<v-clicks>

- **Don't expose stack traces** in production
- **Log errors** with context for debugging
- **Use meaningful error messages** for clients
- **Consistent error format** across your API
- **Include request IDs** for tracing
- **Document error responses** in API docs

</v-clicks>

---

# Summary

<v-clicks>

- Create custom exception classes for your domain
- Use `@ExceptionHandler` for controller-level handling
- Use `@RestControllerAdvice` for global handling
- Return consistent error response DTOs
- Handle validation errors separately
- Log errors appropriately
- Never expose internal details in production

</v-clicks>
