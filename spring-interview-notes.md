# Spring Core, Spring MVC and Spring Boot: Interview Notes for Java Freshers

> **Version note (November 2026):** Examples use **Java 21** and **Spring Boot 4.x** (built on Spring Framework 7). Many company projects still run Boot 3.x. The core concepts are the same, and differences are pointed out where they matter.

## Contents

- [Part 1: Spring Core](#part-1-spring-core)
- [Part 2: Spring MVC](#part-2-spring-mvc)
- [Part 3: Spring Boot](#part-3-spring-boot)
- [Part 4: Mini CRUD Project](#part-4-mini-crud-project)
- [Part 5: Interview Questions and Answers](#part-5-interview-questions-and-answers)
- [Part 6: Last-Day Revision](#part-6-last-day-revision)

**Priority tags:** 🔴 asked very often · 🟡 asked sometimes · 🟢 good to know

---

# Part 1: Spring Core

## 1.1 What is Spring? 🔴

**Spring** is an open-source Java framework for building applications. Its main job is to **create your objects and connect them together**, so you can focus on business logic.

| Module | What it does |
|---|---|
| Core / Beans / Context | IoC container and dependency injection |
| AOP | Adds common logic (logging, security) without touching business code |
| JDBC / ORM / Transactions | Database access |
| Web MVC | Web applications and REST APIs |
| Test | Testing support |

**Advantages:** loose coupling, easier testing, less boilerplate, modular, large ecosystem.

## 1.2 The problem Spring solves 🔴

**Without Spring (tight coupling):**

```java
class OrderService {
    // OrderService decides which class to use and creates it
    private final EmailSender sender = new EmailSender();
}
```

Problems: switching to SMS means editing `OrderService`, and you cannot easily test it with a fake sender.

**With Spring (loose coupling):**

```java
interface MessageSender {
    void send(String message);
}

@Component
class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

@Service
class OrderService {
    private final MessageSender sender;

    OrderService(MessageSender sender) {   // Spring passes the object in
        this.sender = sender;
    }

    void placeOrder() {
        sender.send("Order placed");
    }
}
```

## 1.3 Inversion of Control (IoC) 🔴

**IoC** means the control of creating and managing objects moves from **your code** to the **Spring container**.

**Analogy:** Cooking at home, you buy and prepare everything. At a restaurant, you just ask and it is served to you.

## 1.4 Dependency Injection (DI) 🔴

A **dependency** is an object that another object needs. **DI** means the container **supplies** that object instead of the class creating it.

> **Interview line:** IoC is the principle. DI is the technique Spring uses to achieve it.

### Three types of injection

```java
// 1. Constructor injection (recommended)
@Service
class StudentService {
    private final StudentRepository repo;

    StudentService(StudentRepository repo) {
        this.repo = repo;
    }
}

// 2. Setter injection (for optional dependencies)
@Service
class ReportService {
    private Formatter formatter;

    @Autowired
    void setFormatter(Formatter formatter) {
        this.formatter = formatter;
    }
}

// 3. Field injection (avoid in production code)
@Service
class CourseService {
    @Autowired
    private CourseRepository repo;
}
```

**Why is constructor injection preferred?** 🔴

1. Fields can be `final` (immutable).
2. The object is never half-built; required dependencies must be present.
3. Easy to test: `new StudentService(fakeRepo)`.
4. Circular dependencies show up at startup.
5. A long constructor warns you that the class is doing too much.

> If a class has **only one constructor**, `@Autowired` is **not required** on it.

## 1.5 Spring IoC Container 🔴

The container creates beans, wires them, and manages their lifecycle. It reads your **configuration** (annotations, Java classes, or XML).

| Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Level | Basic container | Extends `BeanFactory` |
| Singleton creation | Lazy (when requested) | Eager (at startup) by default |
| Events, internationalisation | No | Yes |
| Automatic `BeanPostProcessor` registration | No | Yes |
| Used in real projects | Rarely | Almost always |

Common implementations: `AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext`.

```java
@Configuration
@ComponentScan("com.example")
class AppConfig { }

public class Main {
    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(AppConfig.class)) {
            OrderService service = context.getBean(OrderService.class);
            service.placeOrder();
        }
    }
}
```

## 1.6 Spring Beans and configuration 🔴

A **bean** is an object that is created and managed by the Spring container.

### Ways to define a bean

**A. Component scanning** (for your own classes)

```java
@Component
class PdfGenerator { }
```

**B. Java configuration** (for third-party classes, or when creation needs custom code)

```java
@Configuration
class ToolConfig {
    @Bean
    Clock clock() {
        return Clock.systemUTC();
    }
}
```

**C. XML** (old style, seen in legacy projects)

```xml
<bean id="orderService" class="com.example.OrderService">
    <constructor-arg ref="emailSender"/>
</bean>
```

### `@Component` vs `@Bean` 🔴

| `@Component` | `@Bean` |
|---|---|
| Placed on a **class** | Placed on a **method** |
| Found by component scanning | Declared explicitly, usually in `@Configuration` |
| For classes you own | Works for library classes too |

### Stereotype annotations 🔴

| Annotation | Layer | Extra behaviour |
|---|---|---|
| `@Component` | Generic | None |
| `@Service` | Business logic | None (shows intent) |
| `@Repository` | Data access | Translates database exceptions into Spring's `DataAccessException` |
| `@Controller` | Web | Handles requests, usually returns views |
| `@RestController` | Web (REST) | `@Controller` + `@ResponseBody` |

## 1.7 Autowiring and resolving ambiguity 🔴

`@Autowired` injects **by type**. If two beans have the same type, Spring cannot choose and throws `NoUniqueBeanDefinitionException`.

```java
@Component
@Primary                       // default choice
class EmailSender implements MessageSender {
    public void send(String m) { }
}

@Component("sms")
class SmsSender implements MessageSender {
    public void send(String m) { }
}

@Service
class AlertService {
    private final MessageSender sender;

    AlertService(@Qualifier("sms") MessageSender sender) {  // specific choice
        this.sender = sender;
    }
}
```

- `@Qualifier` **wins over** `@Primary`.
- Without either, Spring tries to match the parameter or field **name** with a bean name.
- Optional dependency: `@Autowired(required = false)` or `Optional<T>`.
- Inject all implementations: `List<MessageSender> senders`.

## 1.8 Bean scopes 🔴

| Scope | Instances | Available in |
|---|---|---|
| `singleton` (default) | One per Spring container | All apps |
| `prototype` | New one every time it is requested | All apps |
| `request` | One per HTTP request | Web apps |
| `session` | One per HTTP session | Web apps |
| `application` | One per `ServletContext` | Web apps |
| `websocket` | One per WebSocket session | Web apps |

```java
@Component
@Scope("prototype")
class ShoppingCart { }
```

**Tricky points:**

- Singleton beans are **not automatically thread-safe**. Avoid mutable fields in them.
- Spring singleton means one per **container per bean definition**, unlike the GoF pattern (one per class loader).
- A prototype injected into a singleton is created **only once**. To get a new one each time, use `ObjectProvider`:

```java
@Service
class CheckoutService {
    private final ObjectProvider<ShoppingCart> carts;

    CheckoutService(ObjectProvider<ShoppingCart> carts) {
        this.carts = carts;
    }

    ShoppingCart newCart() {
        return carts.getObject();   // new instance on every call
    }
}
```

## 1.9 Bean lifecycle 🔴

```text
1. Instantiate the bean (constructor)
2. Inject dependencies
3. Aware callbacks (BeanNameAware, ApplicationContextAware ...)
4. BeanPostProcessor: before initialization
5. Initialization: @PostConstruct -> afterPropertiesSet() -> initMethod
6. BeanPostProcessor: after initialization (AOP proxies are usually created here)
7. Bean is ready to use
8. Container shuts down: @PreDestroy -> destroy() -> destroyMethod
```

```java
@Component
class CacheLoader {
    @PostConstruct
    void load() {
        System.out.println("Loading cache");
    }

    @PreDestroy
    void clear() {
        System.out.println("Clearing cache");
    }
}
```

- These annotations come from `jakarta.annotation` (older projects use `javax.annotation`).
- Spring does **not** call destroy methods for **prototype** beans.
- `@Lazy` delays creating a singleton until it is first needed.

## 1.10 Circular dependency 🟡

Bean A needs B, and B needs A. With constructor injection neither can be created, so startup fails.

**Fix:** redesign (best, often by moving shared logic into a third class). `@Lazy` on one dependency is a workaround. Spring Boot blocks circular references by default (since 2.6).

## 1.11 Reading values: `@Value` 🟡

```java
@Component
class AppInfo {
    @Value("${app.name:MyApp}")     // property with a default value
    private String name;

    @Value("#{2 * 5}")              // SpEL (Spring Expression Language)
    private int number;
}
```

`${...}` reads a **property**. `#{...}` evaluates an **expression**.

## 1.12 Aspect-Oriented Programming (AOP) 🔴

Some logic is needed everywhere: logging, security, transactions. These are **cross-cutting concerns**. AOP lets you write them **once**.

| Term | Meaning |
|---|---|
| Aspect | Class holding the common logic |
| Advice | The action to run |
| Join point | A point where advice can run (in Spring: a method execution) |
| Pointcut | Expression that selects join points |
| Weaving | Linking aspects to target objects |
| Proxy | Wrapper object Spring creates around your bean |

**Advice types:** `@Before`, `@After` (always), `@AfterReturning`, `@AfterThrowing`, `@Around`.

```java
@Aspect
@Component
class TimingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    Object measure(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();          // run the real method
        long taken = System.currentTimeMillis() - start;
        System.out.println(pjp.getSignature() + " took " + taken + " ms");
        return result;
    }
}
```

> Dependency: `spring-boot-starter-aspectj` in Boot 4 (`spring-boot-starter-aop` in Boot 3).

**Proxy types:** JDK dynamic proxy (interface-based) and CGLIB (subclass-based). Spring Boot uses CGLIB by default.

## 1.13 Transactions: `@Transactional` 🔴

A transaction means **all steps succeed or none do**.

```java
@Service
class TransferService {
    private final AccountRepository accounts;

    TransferService(AccountRepository accounts) {
        this.accounts = accounts;
    }

    @Transactional
    public void transfer(long fromId, long toId, BigDecimal amount) {
        accounts.withdraw(fromId, amount);
        accounts.deposit(toId, amount);   // failure here undoes the withdrawal
    }
}
```

**Interview traps:**

- By default it rolls back for **unchecked exceptions and errors**, not checked exceptions. Use `rollbackFor = Exception.class` to change that.
- **Self-invocation:** calling a `@Transactional` method from another method of the **same class** skips the proxy, so the annotation has no effect.
- `private` methods are not intercepted.
- Common propagation: `REQUIRED` (default, join or create) and `REQUIRES_NEW` (always a new one).

## 1.14 Design patterns used in Spring 🟡

| Pattern | Where |
|---|---|
| Singleton | Default bean scope |
| Factory | `BeanFactory` |
| Proxy | AOP, transactions |
| Template Method | `JdbcTemplate` |
| Front Controller | `DispatcherServlet` |
| Observer | Application events |

---

# Part 2: Spring MVC

## 2.1 What is Spring MVC? 🔴

Spring's web framework, built on the **Servlet API**, following the **Model-View-Controller** pattern.

| Part | Role | Example |
|---|---|---|
| Model | Data | A list of students |
| View | What the user sees | Thymeleaf or JSP page |
| Controller | Receives requests and coordinates | `StudentController` |

## 2.2 Request flow and `DispatcherServlet` 🔴

`DispatcherServlet` is the **front controller**: every request goes through it first.

```text
Client
  |  1. HTTP request
  v
Filters
  |
  v
DispatcherServlet ---- 2. which method? ----> HandlerMapping
  |
  | 3. call it through HandlerAdapter
  |    (interceptors run before and after)
  v
Controller -> Service -> Repository -> Database
  |
  | 4a. Web page: returns view name -> ViewResolver -> View renders HTML
  | 4b. REST API: returns object -> HttpMessageConverter -> JSON
  v
Client receives the response
```

> **Say it in 30 seconds:** The request reaches `DispatcherServlet`, which asks `HandlerMapping` for the right controller method. A `HandlerAdapter` invokes it. For web pages a `ViewResolver` finds the view. For REST, a message converter (Jackson) turns the object into JSON.

## 2.3 `@Controller` vs `@RestController` 🔴

```java
@Controller
class PageController {
    @GetMapping("/welcome")
    String welcome(Model model) {
        model.addAttribute("name", "Asha");
        return "welcome";   // view name -> templates/welcome.html
    }
}

@RestController
class GreetingController {
    @GetMapping("/api/greet")
    String greet() {
        return "welcome";   // the text itself is the response body
    }
}
```

Thymeleaf view (`src/main/resources/templates/welcome.html`):

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
  <h1 th:text="'Hello ' + ${name}">Hello</h1>
</body>
</html>
```

## 2.4 Mapping annotations 🔴

| Annotation | HTTP method | Use | Idempotent? |
|---|---|---|---|
| `@GetMapping` | GET | Read | Yes |
| `@PostMapping` | POST | Create | No |
| `@PutMapping` | PUT | Replace | Yes |
| `@PatchMapping` | PATCH | Partial update | Not guaranteed |
| `@DeleteMapping` | DELETE | Delete | Yes |

`@RequestMapping` is the general form, often used on the class for a common prefix. **Idempotent** means repeating the request leaves the server in the same state.

## 2.5 Getting data from a request 🔴

```java
@RestController
@RequestMapping("/api/books")
class BookController {

    // GET /api/books/10
    @GetMapping("/{id}")
    String byId(@PathVariable Long id) {
        return "Book " + id;
    }

    // GET /api/books?author=Ravi&page=2
    @GetMapping
    String search(@RequestParam String author,
                  @RequestParam(defaultValue = "1") int page) {
        return author + " page " + page;
    }

    // POST /api/books with a JSON body
    @PostMapping
    Book create(@RequestBody Book book) {
        return book;
    }

    // Read a header
    @GetMapping("/agent")
    String agent(@RequestHeader("User-Agent") String agent) {
        return agent;
    }
}

record Book(String title, String author) { }
```

| Annotation | Reads from |
|---|---|
| `@PathVariable` | URL path: `/books/10` |
| `@RequestParam` | Query string or form field: `?page=2` |
| `@RequestBody` | Request body (JSON to object) |
| `@RequestHeader` | HTTP header |
| `@ModelAttribute` | Form fields bound to an object |

## 2.6 `ResponseEntity` and status codes 🔴

`ResponseEntity` lets you control **status, headers and body**.

```java
@PostMapping
ResponseEntity<Book> create(@RequestBody Book book) {
    return ResponseEntity.status(HttpStatus.CREATED).body(book);
}
```

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized (not logged in) |
| 403 | Forbidden (logged in, not allowed) |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 415 | Unsupported Media Type |
| 500 | Internal Server Error |

## 2.7 Validation 🔴

Add `spring-boot-starter-validation`.

```java
record SignupRequest(
    @NotBlank String name,
    @Email @NotBlank String email,
    @Min(18) int age
) { }

@PostMapping("/signup")
String signup(@Valid @RequestBody SignupRequest request) {
    return "OK";
}
```

If validation fails, `MethodArgumentNotValidException` is thrown and the default response is **400**.

| Annotation | Checks |
|---|---|
| `@NotNull` | Not null |
| `@NotEmpty` | Not null and not empty |
| `@NotBlank` | Text has at least one non-space character |
| `@Size` | Length or size range |
| `@Min`, `@Max` | Number limits |
| `@Pattern` | Regular expression |

## 2.8 Exception handling 🔴

```java
class BookNotFoundException extends RuntimeException {
    BookNotFoundException(Long id) {
        super("Book not found: " + id);
    }
}

@RestControllerAdvice            // applies to all controllers
class GlobalExceptionHandler {

    @ExceptionHandler(BookNotFoundException.class)
    ProblemDetail handleNotFound(BookNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(
                HttpStatus.NOT_FOUND, ex.getMessage());
    }
}
```

- `@ExceptionHandler` inside a controller handles errors for **that controller only**.
- `@ControllerAdvice` / `@RestControllerAdvice` handles them **globally**.
- `ProblemDetail` is a standard error response format.

## 2.9 Filter vs Interceptor 🟡

| Filter | Interceptor |
|---|---|
| Part of the Servlet API | Part of Spring MVC |
| Runs before and after `DispatcherServlet` | Runs around the controller |
| Does not receive the handler method | Receives the handler method |
| Encoding, CORS, security | Logging, permission checks |

```java
class LogInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req,
                             HttpServletResponse res, Object handler) {
        System.out.println("Incoming: " + req.getRequestURI());
        return true;   // false stops the request
    }
}

@Configuration
class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor()).addPathPatterns("/api/**");
    }
}
```

Interceptor methods: `preHandle`, `postHandle`, `afterCompletion`.

## 2.10 CORS 🟡

Browsers block a page from reading responses from a different origin (protocol, domain or port) unless the server allows it.

```java
@CrossOrigin(origins = "http://localhost:3000")
@RestController
class ProductController { }
```

## 2.11 REST basics 🔴

- Use **nouns** in URLs: `/students/5`, not `/getStudent?id=5`.
- Use HTTP methods for actions and correct status codes.
- REST is **stateless**: each request carries all the information needed.

---

# Part 3: Spring Boot

## 3.1 What is Spring Boot? 🔴

Spring Boot is built **on top of Spring** to make setup fast. It gives you:

1. **Starters:** ready-made dependency bundles
2. **Auto-configuration:** sensible setup based on what is on the classpath
3. **Embedded server:** run with `java -jar`
4. **Production features:** health checks and metrics through Actuator

### Spring vs Spring MVC vs Spring Boot 🔴

| Spring | Spring MVC | Spring Boot |
|---|---|---|
| Core framework (IoC, DI, AOP) | Web module of Spring | Tool to build Spring apps quickly |
| Manual configuration | Needs web configuration | Auto-configuration |
| External server for web apps | External server, traditionally | Embedded server |
| Manage dependency versions yourself | Same | Starters manage versions |

> Spring Boot does **not replace** Spring or Spring MVC. It configures them for you.

## 3.2 Creating a project 🟡

Go to **start.spring.io**, choose Maven or Gradle, Java 21, add dependencies, then generate.

```text
src/main/java/com/example/demo/
    DemoApplication.java      <- main class in the root package
    controller/
    service/
    repository/
    model/
src/main/resources/
    application.properties
    static/                   <- CSS, JS, images
    templates/                <- Thymeleaf views
src/test/java/
pom.xml
```

## 3.3 `@SpringBootApplication` 🔴

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

It combines three annotations:

| Annotation | Purpose |
|---|---|
| `@SpringBootConfiguration` | Marks it as a configuration class |
| `@EnableAutoConfiguration` | Turns on auto-configuration |
| `@ComponentScan` | Scans this package and its sub-packages |

> **Common bug:** a controller in a package outside the main class's package is not found, so you get 404.

**What `SpringApplication.run()` does:** prepares the environment (properties, profiles), creates the `ApplicationContext`, registers and creates beans, starts the embedded server, then runs any runners.

## 3.4 Starters 🔴

| Starter | Gives you |
|---|---|
| `spring-boot-starter-webmvc` | Spring MVC, JSON, Tomcat (Boot 3: `spring-boot-starter-web`) |
| `spring-boot-starter-data-jpa` | Spring Data JPA, Hibernate |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-validation` | Bean validation |
| `spring-boot-starter-thymeleaf` | Thymeleaf views |
| `spring-boot-starter-actuator` | Monitoring endpoints |
| `spring-boot-starter-test` | JUnit, Mockito, AssertJ |

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

No `<version>` is needed because the **parent POM** (`spring-boot-starter-parent`) manages versions.

## 3.5 Auto-configuration 🔴

Spring Boot looks at your **classpath, beans and properties**, then configures things automatically.

*Example:* a database driver and JPA are present, so Boot sets up a `DataSource` and related beans.

**How it works:**

1. `@EnableAutoConfiguration` loads candidate classes listed in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
2. Each applies only if its **conditions** are true.

| Condition | Applies when |
|---|---|
| `@ConditionalOnClass` | A class is on the classpath |
| `@ConditionalOnMissingBean` | You have not defined that bean |
| `@ConditionalOnProperty` | A property has a given value |

```java
@AutoConfiguration
@ConditionalOnClass(Clock.class)
class ClockAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    Clock clock() {
        return Clock.systemDefaultZone();
    }
}
```

- **Override:** define your own bean; the default usually backs off.
- **Exclude:** `@SpringBootApplication(exclude = SomeAutoConfiguration.class)`
- **Debug:** run with `--debug` to see what was applied and why.

## 3.6 Configuration files 🔴

`application.properties`:

```properties
server.port=8081
spring.application.name=demo
spring.datasource.url=jdbc:mysql://localhost:3306/college
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

Same idea in `application.yml`:

```yaml
server:
  port: 8081
spring:
  application:
    name: demo
```

**Priority (highest first, simplified):**

1. Command-line arguments (`--server.port=9000`)
2. Java system properties
3. OS environment variables
4. Profile-specific file (`application-dev.properties`)
5. `application.properties`

Files outside the JAR override files inside it.

### `@ConfigurationProperties` 🟡

```properties
app.mail.host=smtp.example.com
app.mail.port=587
```

```java
@ConfigurationProperties(prefix = "app.mail")
record MailProperties(String host, int port) { }
```

Enable it with `@ConfigurationPropertiesScan` on the main class.

| `@Value` | `@ConfigurationProperties` |
|---|---|
| One value at a time | A group of related values |
| Supports SpEL | No SpEL |
| Limited relaxed binding | Full relaxed binding, type-safe |

## 3.7 Profiles 🔴

Different settings for different environments.

Files: `application-dev.properties`, `application-prod.properties`

```properties
spring.profiles.active=dev
```

```java
@Bean
@Profile("dev")
CommandLineRunner sampleData() {
    return args -> System.out.println("Adding sample data");
}
```

Run: `java -jar app.jar --spring.profiles.active=prod`

## 3.8 Embedded server 🔴

- Default is **Tomcat** on port **8080**.
- Jetty is an alternative. (Boot 3 also supported Undertow; Boot 4 removed it.)
- Change the port with `server.port`. Use `0` for a random port.
- To switch server, exclude the Tomcat starter and add the Jetty starter.

**JAR vs WAR:** Boot usually builds an executable JAR containing the server. WAR is used when deploying to an external server.

## 3.9 Actuator 🔴

Endpoints for monitoring a running app.

| Endpoint | Shows |
|---|---|
| `/actuator/health` | Up or down |
| `/actuator/info` | App information |
| `/actuator/metrics` | Metrics |
| `/actuator/env` | Environment properties |
| `/actuator/beans` | All beans |

Only `health` is exposed over HTTP by default.

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

Do not expose sensitive endpoints publicly without security.

## 3.10 DevTools, runners and logging 🟡

- **DevTools** (`spring-boot-devtools`): automatic restart during development.
- **`CommandLineRunner` / `ApplicationRunner`:** run code once after startup. The first gets raw `String` arguments, the second gets parsed `ApplicationArguments`.
- **Logging:** SLF4J with Logback by default.

```java
private static final Logger log = LoggerFactory.getLogger(DemoApplication.class);
// log.info("Started user {}", userId);
```

```properties
logging.level.com.example=DEBUG
```

## 3.11 Calling other APIs 🟢

| Client | Style |
|---|---|
| `RestClient` | Modern, synchronous (preferred) |
| `WebClient` | Reactive, non-blocking |
| `RestTemplate` | Older; being phased out (Boot 4.2 deprecates its Boot support) |

## 3.12 Testing 🟡

| Annotation | Loads |
|---|---|
| `@SpringBootTest` | The whole application context |
| `@WebMvcTest` | Web layer only |
| `@DataJpaTest` | JPA layer only |
| `@MockitoBean` | Replaces a bean with a mock (older code: `@MockBean`) |

> In Boot 4, `@WebMvcTest` needs `spring-boot-starter-webmvc-test` and `@DataJpaTest` needs `spring-boot-starter-data-jpa-test`.

Plain unit test, no Spring needed:

```java
@ExtendWith(MockitoExtension.class)
class StudentServiceTest {
    @Mock StudentRepository repo;
    @InjectMocks StudentService service;

    @Test
    void returnsAllStudents() {
        when(repo.findAll()).thenReturn(List.of());
        assertTrue(service.findAll().isEmpty());
    }
}
```

## 3.13 Version awareness 🟢

| Version | Key points |
|---|---|
| Boot 2.x | `javax.*` packages |
| Boot 3.x | Java 17 minimum, `jakarta.*` packages, Spring Framework 6 |
| Boot 4.x | Java 17 minimum, Spring Framework 7, more modular starters, Jackson 3 by default |

---

# Part 4: Mini CRUD Project

Interviewers often ask you to **explain the layers**.

```text
Controller (HTTP) -> Service (business rules) -> Repository (data) -> Database
```

Dependencies: `spring-boot-starter-webmvc`, `spring-boot-starter-data-jpa`, `spring-boot-starter-validation`, and `h2` (runtime scope). Imports are omitted for brevity; each public type goes in its own file.

**Entity**

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String name;

    private String course;

    public Long getId() { return id; }
    public String getName() { return name; }
    public String getCourse() { return course; }
    public void setName(String name) { this.name = name; }
    public void setCourse(String course) { this.course = course; }
}
```

**Repository** (Spring Data creates the implementation)

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    List<Student> findByCourse(String course);   // query built from method name
}
```

**Service**

```java
@Service
public class StudentService {
    private final StudentRepository repo;

    public StudentService(StudentRepository repo) {
        this.repo = repo;
    }

    public List<Student> findAll() {
        return repo.findAll();
    }

    public Student findById(Long id) {
        return repo.findById(id)
                   .orElseThrow(() -> new StudentNotFoundException(id));
    }

    public Student create(Student student) {
        return repo.save(student);
    }

    @Transactional
    public Student update(Long id, Student data) {
        Student existing = findById(id);
        existing.setName(data.getName());
        existing.setCourse(data.getCourse());
        return existing;      // changes are saved when the transaction commits
    }

    public void delete(Long id) {
        repo.delete(findById(id));
    }
}
```

**Controller**

```java
@RestController
@RequestMapping("/api/students")
public class StudentController {
    private final StudentService service;

    public StudentController(StudentService service) {
        this.service = service;
    }

    @GetMapping
    public List<Student> all() {
        return service.findAll();
    }

    @GetMapping("/{id}")
    public Student one(@PathVariable Long id) {
        return service.findById(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Student create(@Valid @RequestBody Student student) {
        return service.create(student);
    }

    @PutMapping("/{id}")
    public Student update(@PathVariable Long id,
                          @Valid @RequestBody Student student) {
        return service.update(id, student);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}
```

**Exception and handler**

```java
public class StudentNotFoundException extends RuntimeException {
    public StudentNotFoundException(Long id) {
        super("Student not found: " + id);
    }
}
```

```java
@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(StudentNotFoundException.class)
    public ProblemDetail notFound(StudentNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }
}
```

> In real projects, accept and return **DTOs** instead of entities. This hides internal fields and stops clients from setting values they should not.

---

# Part 5: Interview Questions and Answers

## Spring Core

**1. What is Spring?**
An open-source Java framework whose core is an IoC container that creates and connects objects.

**2. Difference between IoC and DI?**
IoC is the principle of giving object control to the container. DI is how Spring does it, by supplying dependencies.

**3. What is a bean?**
An object created and managed by the Spring container.

**4. Which injection type do you prefer?**
Constructor injection: immutable fields, mandatory dependencies, easy testing.

**5. `BeanFactory` vs `ApplicationContext`?**
`ApplicationContext` extends `BeanFactory`, creates singletons eagerly, and adds events and internationalisation.

**6. What is the default bean scope?**
Singleton.

**7. Are singleton beans thread-safe?**
Not automatically. Keep them stateless or handle synchronisation.

**8. Explain the bean lifecycle.**
Instantiate, inject, aware callbacks, pre-processing, initialise, post-processing, use, destroy.

**9. Two beans of the same type exist. What happens?**
Spring tries name matching; if that fails, it throws `NoUniqueBeanDefinitionException`. Use `@Qualifier` or `@Primary`.

**10. What is AOP?**
Separating cross-cutting concerns such as logging from business logic, using proxies in Spring.

**11. When does `@Transactional` not work?**
On self-invocation, on private methods, or on objects not managed by Spring. A checked exception also does not roll back by default.

## Spring MVC

**12. What is `DispatcherServlet`?**
The front controller that receives requests and delegates them to controllers.

**13. `@Controller` vs `@RestController`?**
`@RestController` writes return values to the response body; `@Controller` normally returns view names.

**14. `@RequestParam` vs `@PathVariable`?**
Query or form parameter versus a value inside the URL path.

**15. PUT vs PATCH vs POST?**
PUT replaces a resource, PATCH changes part of it, POST usually creates one.

**16. How do you handle exceptions globally?**
`@RestControllerAdvice` with `@ExceptionHandler` methods.

**17. How is an object converted to JSON?**
An `HttpMessageConverter` uses the Jackson library.

**18. What is a `ViewResolver`?**
It maps a view name to an actual view such as a Thymeleaf template.

## Spring Boot

**19. Why Spring Boot?**
Less configuration, starters, embedded server and production-ready features.

**20. What does `@SpringBootApplication` contain?**
`@SpringBootConfiguration`, `@EnableAutoConfiguration` and `@ComponentScan`.

**21. How does auto-configuration work?**
Boot loads listed configuration classes and applies those whose conditions match the classpath, beans and properties.

**22. What is a starter?**
A dependency that brings a tested set of related libraries.

**23. How do you change the port?**
`server.port=9090`

**24. What are profiles?**
Named sets of configuration for environments such as dev and prod.

**25. What is Actuator?**
Monitoring and management endpoints such as health and metrics.

**26. How do you connect to a database?**
Add the JPA starter and driver, set `spring.datasource.*` properties, create entities and repositories.

**27. What does `spring.jpa.hibernate.ddl-auto` do?**
Controls schema handling: `none`, `validate`, `update`, `create`, `create-drop`. Avoid `update` and `create` in production.

## Scenario questions

| Scenario | What to check or say |
|---|---|
| API returns 404 | URL, HTTP mapping, context path, package covered by component scan |
| `NoSuchBeanDefinitionException` | Missing stereotype annotation, wrong package, inactive profile |
| Port 8080 already in use | Stop the other process or change `server.port` |
| `@RequestBody` fields are null | JSON names do not match, or missing setters / constructor |
| 415 error | Send `Content-Type: application/json` |
| Data not rolled back | Checked exception, self-invocation, or exception caught and hidden |
| App fails on database setup | JPA or JDBC added without a valid datasource configuration |

---

# Part 6: Last-Day Revision

## Annotation cheat sheet

| Area | Annotations |
|---|---|
| Core | `@Component` `@Service` `@Repository` `@Configuration` `@Bean` `@ComponentScan` |
| Wiring | `@Autowired` `@Qualifier` `@Primary` `@Lazy` `@Scope` `@Value` `@Profile` |
| Lifecycle | `@PostConstruct` `@PreDestroy` |
| AOP | `@Aspect` `@Before` `@After` `@Around` |
| Web | `@Controller` `@RestController` `@RequestMapping` `@GetMapping` `@PostMapping` |
| Request data | `@PathVariable` `@RequestParam` `@RequestBody` `@RequestHeader` |
| Errors and validation | `@ExceptionHandler` `@RestControllerAdvice` `@Valid` |
| Boot | `@SpringBootApplication` `@ConfigurationProperties` |
| Data | `@Entity` `@Id` `@GeneratedValue` `@Transactional` |

## Checklist: can you explain these without looking?

- [ ] IoC and DI, with an example
- [ ] Three injection types and why constructor injection
- [ ] Bean scopes and lifecycle
- [ ] `@Component` vs `@Bean`
- [ ] `@Qualifier` vs `@Primary`
- [ ] AOP terms
- [ ] `@Transactional` pitfalls
- [ ] `DispatcherServlet` request flow (draw it)
- [ ] `@Controller` vs `@RestController`
- [ ] Global exception handling
- [ ] Spring vs Spring Boot
- [ ] Auto-configuration and starters
- [ ] Profiles and property priority
- [ ] Your own project, layer by layer

## Tips for freshers

1. Answer in this order: **definition, why it is needed, small example**.
2. Build the CRUD project yourself and test it with Postman or `curl`.
3. Be ready to explain every annotation in your resume project.
4. If you do not know something, say what you do know and how you would find out.
