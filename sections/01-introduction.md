# What is Spring?

Spring is a comprehensive **framework** for building Java applications

<v-clicks>

- **Lightweight** - Minimal overhead
- **Modular** - Use only what you need
- **Enterprise-ready** - Built for production
- **Community-driven** - Large ecosystem

</v-clicks>

---

# The Problem Spring Solves

### Before Spring:

```java
// Manual dependency creation
public class OrderService {
    private PaymentService paymentService;
    private InventoryService inventoryService;

    public OrderService() {
        this.paymentService = new PaymentService();
        this.inventoryService = new InventoryService();
    }
}
```

<v-click>

**Problems:**
- Tight coupling
- Hard to test
- Difficult to maintain

</v-click>

---

# What is Spring Boot?

Spring Boot is an **opinionated** framework built on top of Spring

<v-clicks>

- **Convention over Configuration** - Sensible defaults
- **Embedded Server** - No external server needed
- **Auto-configuration** - Smart defaults
- **Production-ready** - Health checks, metrics out of the box
- **No XML** - Java-based configuration

</v-clicks>

---

# Spring vs Spring Boot

| Aspect | Spring | Spring Boot |
|--------|--------|-------------|
| Configuration | Manual, verbose | Auto-configured |
| Server | External (Tomcat, etc.) | Embedded |
| Dependencies | Manual management | Starter POMs |
| Setup Time | Hours | Minutes |
| XML Config | Often required | Not needed |

---

# Why Spring Boot?

<div class="grid grid-cols-2 gap-4">
<div>

### Developer Experience
- Quick project setup
- Less boilerplate code
- Hot reload support
- Great IDE support

</div>
<div>

### Production Ready
- Health indicators
- Metrics & monitoring
- Externalized configuration
- Easy deployment

</div>
</div>

---

# Spring Boot Features

```
                    Spring Boot
                         |
    +---------+---------+---------+---------+
    |         |         |         |         |
Auto-Config  Starters  Actuator  DevTools  CLI
```

<v-clicks>

- **Auto-configuration** - Automatic bean configuration
- **Starters** - Dependency management simplified
- **Actuator** - Production monitoring
- **DevTools** - Development productivity
- **CLI** - Command-line interface

</v-clicks>

---

# Real World Usage

<div class="grid grid-cols-3 gap-4 text-center">
<div>

### Netflix
Microservices platform

</div>
<div>

### Alibaba
E-commerce backend

</div>
<div>

### LinkedIn
Social platform services

</div>
</div>

<br>

> "Spring Boot has become the de facto standard for building Java microservices"

---

# Prerequisites for This Course

<v-clicks>

- **Java Knowledge** - OOP concepts, collections, streams
- **Basic HTTP** - Request/response, status codes
- **JSON** - Data format basics
- **IDE Familiarity** - IntelliJ IDEA or VS Code

</v-clicks>

