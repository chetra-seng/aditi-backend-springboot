# Building REST APIs

## Controllers, Services, and HTTP

---

# What is REST?

**RE**presentational **S**tate **T**ransfer

<v-clicks>

- Architectural style for web services
- Uses HTTP methods as verbs
- Resources identified by URLs
- Stateless communication
- JSON/XML for data exchange

</v-clicks>

---

# REST Principles

| Principle | Description |
|-----------|-------------|
| Stateless | No client context stored on server |
| Client-Server | Separation of concerns |
| Cacheable | Responses can be cached |
| Uniform Interface | Consistent API design |
| Layered System | Multiple layers possible |

---

# HTTP Methods

| Method | Purpose | CRUD | Idempotent |
|--------|---------|------|------------|
| GET | Retrieve resource | Read | Yes |
| POST | Create resource | Create | No |
| PUT | Update/Replace resource | Update | Yes |
| PATCH | Partial update | Update | No |
| DELETE | Remove resource | Delete | Yes |

---

# What is Idempotent?

Same request, same result (no matter how many times)

```bash
# Idempotent - same result every time
GET /users/1      # Always returns user 1
PUT /users/1      # Always replaces user 1
DELETE /users/1   # First deletes, then "already deleted"

# NOT Idempotent - result changes
POST /users       # Creates new user each time
```

---

# HTTP Status Codes

| Range | Category | Examples |
|-------|----------|----------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 404 Not Found, 422 Unprocessable |
| 5xx | Server Error | 500 Internal Error, 503 Unavailable |

---

# Common Status Codes in Detail

| Code | When to Use |
|------|-------------|
| 200 OK | GET success, PUT/PATCH success |
| 201 Created | POST success (resource created) |
| 204 No Content | DELETE success, PUT with no body |
| 400 Bad Request | Malformed request syntax |
| 404 Not Found | Resource doesn't exist |
| 409 Conflict | Duplicate resource |
| 500 Internal Error | Unexpected server error |

---

# Layered Architecture

```
┌─────────────────────────────────────────┐
│           Controller Layer              │
│    (HTTP handling, request/response)    │
├─────────────────────────────────────────┤
│            Service Layer                │
│         (Business logic)                │
├─────────────────────────────────────────┤
│          Repository Layer               │
│           (Data access)                 │
└─────────────────────────────────────────┘
```

---

# Why Layers?

<v-clicks>

- **Separation of Concerns** - Each layer has one job
- **Testability** - Test each layer independently
- **Maintainability** - Change one layer without affecting others
- **Reusability** - Services can be used by multiple controllers

</v-clicks>

---

# The Model Class

```java
public class User {
    private Long id;
    private String name;
    private String email;

    // Default constructor
    public User() {}

    // Constructor with fields
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters and setters...
}
```

---

# Model with Lombok

```java
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data  // Generates getters, setters, toString, equals, hashCode
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
    private String email;
}
```

---

# The Service Layer

```java
@Service
public class UserService {

    private final Map<Long, User> users = new HashMap<>();
    private Long nextId = 1L;

    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }

    public Optional<User> findById(Long id) {
        return Optional.ofNullable(users.get(id));
    }

    // continued...
}
```

---

# Service Layer (continued)

```java
    public User create(String name, String email) {
        User user = new User(nextId++, name, email);
        users.put(user.getId(), user);
        return user;
    }

    public Optional<User> update(Long id, String name, String email) {
        return findById(id).map(user -> {
            user.setName(name);
            user.setEmail(email);
            return user;
        });
    }

    public boolean delete(Long id) {
        return users.remove(id) != null;
    }
}
```

---

# @RestController

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
}
```

---

# @RestController Explained

```java
@RestController
// Equivalent to:
@Controller
@ResponseBody
```

<v-click>

- `@Controller` - Marks as Spring MVC controller
- `@ResponseBody` - Return value serialized to JSON
- Combined = REST API controller

</v-click>

---

# Request Mapping Annotations

```java
@GetMapping("/users")      // GET /users
@PostMapping("/users")     // POST /users
@PutMapping("/users/{id}") // PUT /users/123
@PatchMapping("/users/{id}") // PATCH /users/123
@DeleteMapping("/users/{id}") // DELETE /users/123
```

<v-click>

Alternative syntax:
```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
```

</v-click>

---

# @PathVariable

Extract values from URL path:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}

// GET /users/123 → id = 123
```

<v-click>

With custom name:
```java
@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(@PathVariable("userId") Long userId,
                      @PathVariable("orderId") Long orderId) {
    // ...
}
```

</v-click>

---

# @RequestParam

Extract query parameters:

```java
@GetMapping("/users")
public List<User> searchUsers(
    @RequestParam String name,
    @RequestParam(required = false) Integer age,
    @RequestParam(defaultValue = "0") int page) {
    // GET /users?name=John&age=25&page=0
    return userService.search(name, age, page);
}
```

---

# @PathVariable vs @RequestParam

```java
// PathVariable - identifies a specific resource
GET /users/123           // Get user with ID 123
GET /orders/456/items/789  // Get specific item in order

// RequestParam - filters or modifies the query
GET /users?name=John     // Search users by name
GET /users?page=2&size=10  // Pagination
GET /products?category=electronics&sort=price
```

---

# @RequestBody

Deserialize JSON request body:

```java
@PostMapping("/users")
public User createUser(@RequestBody CreateUserRequest request) {
    return userService.create(request.getName(), request.getEmail());
}
```

```json
// POST /users
// Content-Type: application/json
{
    "name": "John Doe",
    "email": "john@example.com"
}
```

---

# Request DTO

```java
public class CreateUserRequest {
    private String name;
    private String email;

    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

Or with Lombok:
```java
@Data
public class CreateUserRequest {
    private String name;
    private String email;
}
```

---

# Using Java Records (Java 17+)

```java
public record CreateUserRequest(
    String name,
    String email
) {}

public record UserResponse(
    Long id,
    String name,
    String email
) {}
```

Records are:
- Immutable
- Auto-generate constructor, getters, equals, hashCode, toString

---

# ResponseEntity

Full control over HTTP response:

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(user -> ResponseEntity.ok(user))
        .orElse(ResponseEntity.notFound().build());
}
```

---

# ResponseEntity Examples

```java
// 200 OK with body
return ResponseEntity.ok(user);

// 201 Created with location header
URI location = URI.create("/api/users/" + user.getId());
return ResponseEntity.created(location).body(user);

// 204 No Content
return ResponseEntity.noContent().build();

// 400 Bad Request
return ResponseEntity.badRequest().body(error);

// 404 Not Found
return ResponseEntity.notFound().build();
```

---

# ResponseEntity with Custom Headers

```java
return ResponseEntity
    .status(HttpStatus.OK)
    .header("X-Custom-Header", "value")
    .header("X-Request-Id", requestId)
    .body(data);
```

---

# Complete CRUD Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAll() {
        return userService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getById(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    // continued...
}
```

---

# CRUD Controller (continued)

```java
    @PostMapping
    public ResponseEntity<User> create(@RequestBody CreateUserRequest request) {
        User created = userService.create(request.getName(), request.getEmail());
        URI location = URI.create("/api/users/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> update(@PathVariable Long id,
                                       @RequestBody CreateUserRequest request) {
        return userService.update(id, request.getName(), request.getEmail())
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (userService.delete(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
```

---

# @RequestHeader

Access HTTP headers:

```java
@GetMapping("/users")
public List<User> getUsers(
    @RequestHeader("Accept-Language") String language,
    @RequestHeader(value = "X-Request-Id", required = false) String requestId) {
    // ...
}
```

---

# Hands-On: Task API

Let's build a Task Management API!

```
GET    /api/tasks          - List all tasks
GET    /api/tasks/{id}     - Get task by ID
POST   /api/tasks          - Create new task
PUT    /api/tasks/{id}     - Update task
DELETE /api/tasks/{id}     - Delete task
PATCH  /api/tasks/{id}/complete - Mark as complete
```

---

# Task Model

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Task {
    private Long id;
    private String title;
    private String description;
    private boolean completed;
    private LocalDateTime createdAt;

    public Task(Long id, String title, String description) {
        this.id = id;
        this.title = title;
        this.description = description;
        this.completed = false;
        this.createdAt = LocalDateTime.now();
    }
}
```

---

# Task Request DTO

```java
@Data
public class TaskRequest {
    private String title;
    private String description;
}
```

Or as record:
```java
public record TaskRequest(
    String title,
    String description
) {}
```

---

# TaskService

```java
@Service
public class TaskService {

    private final Map<Long, Task> tasks = new ConcurrentHashMap<>();
    private final AtomicLong nextId = new AtomicLong(1);

    public List<Task> findAll() {
        return new ArrayList<>(tasks.values());
    }

    public Optional<Task> findById(Long id) {
        return Optional.ofNullable(tasks.get(id));
    }

    public Task create(String title, String description) {
        Task task = new Task(nextId.getAndIncrement(), title, description);
        tasks.put(task.getId(), task);
        return task;
    }
    // ...
}
```

---

# TaskService (continued)

```java
    public Optional<Task> update(Long id, String title, String description) {
        return findById(id).map(task -> {
            task.setTitle(title);
            task.setDescription(description);
            return task;
        });
    }

    public Optional<Task> complete(Long id) {
        return findById(id).map(task -> {
            task.setCompleted(true);
            return task;
        });
    }

    public boolean delete(Long id) {
        return tasks.remove(id) != null;
    }
}
```

---

# TaskController

```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    @GetMapping
    public List<Task> getAllTasks() {
        return taskService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Task> getTask(@PathVariable Long id) {
        return taskService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    // ...
}
```

---

# TaskController (continued)

```java
    @PostMapping
    public ResponseEntity<Task> createTask(@RequestBody TaskRequest request) {
        Task task = taskService.create(request.getTitle(), request.getDescription());
        URI location = URI.create("/api/tasks/" + task.getId());
        return ResponseEntity.created(location).body(task);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Task> updateTask(@PathVariable Long id,
                                           @RequestBody TaskRequest request) {
        return taskService.update(id, request.getTitle(), request.getDescription())
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PatchMapping("/{id}/complete")
    public ResponseEntity<Task> completeTask(@PathVariable Long id) {
        return taskService.complete(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
        return taskService.delete(id)
            ? ResponseEntity.noContent().build()
            : ResponseEntity.notFound().build();
    }
```

---

# Testing with curl

```bash
# Create a task
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn Spring Boot","description":"Complete the course"}'

# Get all tasks
curl http://localhost:8080/api/tasks

# Get single task
curl http://localhost:8080/api/tasks/1

# Complete a task
curl -X PATCH http://localhost:8080/api/tasks/1/complete

# Delete a task
curl -X DELETE http://localhost:8080/api/tasks/1
```

---

# Testing with HTTPie

```bash
# Install: brew install httpie (mac) or pip install httpie

# Create a task
http POST :8080/api/tasks title="Learn Spring" description="Complete course"

# Get all tasks
http :8080/api/tasks

# Get single task
http :8080/api/tasks/1

# Complete a task
http PATCH :8080/api/tasks/1/complete

# Delete a task
http DELETE :8080/api/tasks/1
```

---

# JSON Response Example

```json
// GET /api/tasks/1
{
    "id": 1,
    "title": "Learn Spring Boot",
    "description": "Complete the course",
    "completed": false,
    "createdAt": "2024-01-15T10:30:00"
}

// GET /api/tasks
[
    { "id": 1, "title": "Learn Spring Boot", ... },
    { "id": 2, "title": "Build REST API", ... }
]
```

---

# API Versioning

```java
// URL versioning (most common)
@RequestMapping("/api/v1/users")
@RequestMapping("/api/v2/users")

// Pros: Clear, cacheable, easy to test
// Cons: URL pollution
```

<v-click>

Other approaches:
```java
// Header versioning
@GetMapping(headers = "X-API-VERSION=1")

// Query parameter
@GetMapping(params = "version=1")
```

</v-click>

---

# Filtering with @RequestParam

```java
@GetMapping
public List<Task> getTasks(
    @RequestParam(required = false) Boolean completed,
    @RequestParam(required = false) String search) {

    return taskService.findAll().stream()
        .filter(t -> completed == null || t.isCompleted() == completed)
        .filter(t -> search == null ||
                     t.getTitle().toLowerCase().contains(search.toLowerCase()))
        .toList();
}
```

```bash
GET /api/tasks?completed=true
GET /api/tasks?search=spring
GET /api/tasks?completed=false&search=learn
```

---

# Content Negotiation

Spring handles JSON by default (via Jackson).

```java
@GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
public List<User> getUsers() {
    return userService.findAll();
}
```

Jackson automatically:
- Serializes Java objects to JSON
- Deserializes JSON to Java objects

---

# Customizing JSON Output

```java
@Data
public class User {
    private Long id;
    private String name;

    @JsonProperty("email_address")  // Custom JSON key
    private String email;

    @JsonIgnore  // Exclude from JSON
    private String password;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm")
    private LocalDateTime createdAt;
}
```

---

# Common Mistakes to Avoid

<v-clicks>

1. **Mixing concerns** - Business logic in controller
2. **Not using DTOs** - Exposing internal models
3. **Wrong HTTP methods** - POST for updates, GET for deletions
4. **Wrong status codes** - 200 for everything
5. **Not handling errors** - Returning 500 for everything
6. **Hardcoding IDs** - Using path for auto-generated IDs in POST

</v-clicks>

---

# REST Best Practices

<v-clicks>

- Use **nouns** for resources: `/users` not `/getUsers`
- Use **plural** names: `/users` not `/user`
- Use **HTTP methods** correctly
- Return **appropriate status codes**
- Use **DTOs** for request/response
- Keep **controllers thin**, logic in services
- **Document** your API

</v-clicks>

---

# Summary

<v-clicks>

- `@RestController` = `@Controller` + `@ResponseBody`
- HTTP methods: GET, POST, PUT, PATCH, DELETE
- `@PathVariable` for URL path parameters
- `@RequestParam` for query parameters
- `@RequestBody` for JSON request body
- `ResponseEntity` for full response control
- Layered architecture: Controller → Service → Repository
- Use DTOs to separate API from internal models

</v-clicks>

---

# Exercise

Build a **Product API** with these endpoints:

```
GET    /api/products           - List all products
GET    /api/products/{id}      - Get product by ID
POST   /api/products           - Create product
PUT    /api/products/{id}      - Update product
DELETE /api/products/{id}      - Delete product
GET    /api/products?category=X - Filter by category
```

Product fields: id, name, description, price, category
