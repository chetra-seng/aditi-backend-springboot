# Environment Setup

## Tools & Installation

---

# Required Tools

<v-clicks>

1. **Java Development Kit (JDK)** - Version 17 or higher
2. **IDE** - IntelliJ IDEA (recommended) or VS Code
3. **Build Tool** - Maven or Gradle
4. **API Testing** - Postman or curl

</v-clicks>

---

# Installing JDK 17+

### Windows
```bash
# Using winget
winget install Oracle.JDK.17

# Or download from https://adoptium.net/
```

### macOS
```bash
# Using Homebrew
brew install openjdk@17
```

### Linux
```bash
# Ubuntu/Debian
sudo apt install openjdk-17-jdk

# Fedora
sudo dnf install java-17-openjdk-devel
```

---

# Verify Java Installation

```bash
java -version
```

Expected output:
```
openjdk version "17.0.x" 2023-xx-xx
OpenJDK Runtime Environment (build 17.0.x+xx)
OpenJDK 64-Bit Server VM (build 17.0.x+xx, mixed mode)
```

<v-click>

Also verify `JAVA_HOME`:
```bash
echo $JAVA_HOME
# Should point to your JDK installation
```

</v-click>

---

# IDE Setup - IntelliJ IDEA

### Recommended Plugins:
- Spring Boot (built-in with Ultimate)
- Lombok
- SonarLint
- Rainbow Brackets

### Settings to Configure:
- Enable annotation processing
- Set Java SDK to 17
- Configure code style

---

# IDE Setup - VS Code

### Required Extensions:

```
Extension Pack for Java
Spring Boot Extension Pack
Lombok Annotations Support
```

<v-click>

### settings.json
```json
{
  "java.configuration.runtimes": [
    {
      "name": "JavaSE-17",
      "path": "/path/to/jdk-17"
    }
  ]
}
```

</v-click>

---

# Spring Initializr

The easiest way to create a Spring Boot project

<div class="text-center my-8">

### https://start.spring.io

</div>

<v-clicks>

- Select project metadata
- Choose dependencies
- Generate and download
- Import into IDE

</v-clicks>

---

# Creating Your First Project

### Using Spring Initializr:

```yaml
Project: Maven
Language: Java
Spring Boot: 3.2.x
Group: com.example
Artifact: demo
Packaging: Jar
Java: 17
```

---

# Essential Dependencies

For this course, add these starters:

```xml
<dependencies>
    <!-- Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- MapStruct -->
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>1.5.5.Final</version>
    </dependency>

    <!-- DevTools -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

# MapStruct Annotation Processor

Add to `pom.xml` build section (required for code generation):

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                        <version>1.18.30</version>
                    </path>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>1.5.5.Final</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

# Project Import

### IntelliJ IDEA:
1. File → Open
2. Select the project folder
3. Trust the project
4. Wait for indexing

### VS Code:
1. File → Open Folder
2. Select project folder
3. Java extension will detect Maven/Gradle

---

# Running Your First App

### From IDE:
- Click the green play button on main class

### From Terminal:
```bash
# Maven
./mvnw spring-boot:run

# Gradle
./gradlew bootRun
```

---

# Verify It Works

After starting, you should see:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.x)

Started DemoApplication in 2.345 seconds
```

<v-click>

Visit: **http://localhost:8080**

(You'll see a "Whitelabel Error Page" - that's expected!)

</v-click>

---

# Hello World Endpoint

Create your first endpoint:

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

---

# Test Your Endpoint

```bash
curl http://localhost:8080/hello
```

Response:
```
Hello, Spring Boot!
```

<v-click>

**Congratulations!** You've created your first Spring Boot application!

</v-click>

---

# Development Workflow

```
Edit Code → Save → Auto-restart → Test
```

With DevTools enabled:
- Application auto-restarts on changes
- LiveReload support for browser refresh
- Fast feedback loop

---

# Troubleshooting Common Issues

| Problem | Solution |
|---------|----------|
| Port 8080 in use | Change `server.port` in application.properties |
| Java version mismatch | Check `JAVA_HOME` and pom.xml |
| Dependencies not found | Run `mvn clean install` |
| IDE not recognizing Spring | Reimport Maven project |
