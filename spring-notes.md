# Spring Framework

## Beginner-Friendly Notes for Java Freshers
------------------------------------------------------------------------

## Table of Contents

1.  [Introduction to Spring](#1-introduction-to-spring)
2.  [Why Spring?](#2-why-spring)
3.  [Spring Framework Architecture](#3-spring-framework-architecture)
4.  [Spring Core: IoC and Dependency
    Injection](#4-spring-core-ioc-and-dependency-injection)
5.  [Spring Beans](#5-spring-beans)
6.  [Bean Scopes](#6-bean-scopes)
7.  [Spring Container and
    ApplicationContext](#7-spring-container-and-applicationcontext)
8.  [Configuration in Spring](#8-configuration-in-spring)
9.  [Spring Annotations](#9-important-spring-annotations)
10. [Spring MVC](#10-spring-mvc)
11. [Spring MVC Request
    Processing](#11-spring-mvc-request-processing-flow)
12. [Controllers, Model and Views](#12-controllers-model-and-views)
13. [REST APIs with Spring MVC](#13-rest-apis-with-spring-mvc)
14. [Request Mapping and HTTP
    Methods](#14-request-mapping-and-http-methods)
15. [Validation and Exception
    Handling](#15-validation-and-exception-handling)
16. [Spring Boot](#16-spring-boot)
17. [Spring Boot Auto-Configuration](#17-spring-boot-auto-configuration)
18. [Spring Boot Project Structure](#18-spring-boot-project-structure)
19. [Configuration Properties and
    Profiles](#19-configuration-properties-and-profiles)
20. [Spring Core vs Spring MVC vs Spring
    Boot](#20-spring-core-vs-spring-mvc-vs-spring-boot)
21. [Typical Spring Boot REST
    Application](#21-typical-spring-boot-rest-application)
22. [Common Interview Questions and
    Answers](#22-common-interview-questions-and-answers)
23. [Important Differences for
    Interviews](#23-important-differences-for-interviews)
24. [Fresher Interview Revision
    Checklist](#24-fresher-interview-revision-checklist)
25. [Glossary](#25-glossary)

------------------------------------------------------------------------

# 1. Introduction to Spring

## What is Spring?

Spring is a Java framework used to develop applications. It provides
infrastructure and reusable features so developers can focus more on
application business logic instead of writing repetitive supporting
code.

Spring can be used to build: - Standalone Java applications - Web
applications - RESTful web services - Enterprise applications -
Applications that communicate with databases - Applications using
messaging and other integrations

Spring is not a programming language. It is a framework built mainly for
Java application development.

## What is a framework?

A framework is a reusable structure that provides common functionality
and defines how parts of an application can work together.

In a framework, developers usually write application-specific code and
the framework calls that code at appropriate times. This is commonly
described as **Inversion of Control (IoC)**.

## Spring Framework and Spring Boot

-   **Spring Framework** provides core features such as IoC, Dependency
    Injection, transaction support, and Spring MVC.
-   **Spring Boot** builds on the Spring Framework and makes it easier
    to create, configure, run, and deploy Spring applications.

Spring Boot does not replace Spring Framework. It uses Spring Framework
and reduces the amount of setup normally required.

------------------------------------------------------------------------

# 2. Why Spring?

Before Spring, Java enterprise applications often required developers to
write substantial configuration and infrastructure code.

Common challenges included: - Creating and connecting objects manually -
Managing object dependencies - Writing repetitive setup code - Managing
transactions - Integrating web technologies - Configuring application
components

Spring helps address these challenges.

## Main benefits

### 2.1 Loose coupling

Classes depend on abstractions or externally supplied dependencies
rather than directly creating every required object.

Loose coupling makes code easier to: - Test - Maintain - Replace -
Extend

### 2.2 Dependency Injection

Spring can create objects and provide the dependencies they need.

### 2.3 Modular design

Spring contains modules for different concerns, such as Core, Web, Data
Access, Testing, and Security integrations.

### 2.4 Testability

Dependencies can be supplied or replaced during testing. This makes unit
testing easier.

### 2.5 Less repetitive infrastructure code

Spring provides common infrastructure for tasks such as transaction
management and web request handling.

------------------------------------------------------------------------

# 3. Spring Framework Architecture

Spring is modular. An application does not need to use every Spring
module.

A simplified view:

``` text
Spring Framework
|
+-- Core Container
|   +-- Core
|   +-- Beans
|   +-- Context
|   +-- Expression Language (SpEL)
|
+-- AOP and Instrumentation
|
+-- Data Access / Integration
|   +-- JDBC support
|   +-- Transactions
|   +-- ORM integration
|
+-- Web
|   +-- Spring Web
|   +-- Spring MVC
|   +-- WebSocket support
|
+-- Testing
```

## 3.1 Core Container

The Core Container is responsible for fundamental Spring features.

Important parts:

  Module    Purpose
  --------- ---------------------------------------------------
  Core      Fundamental utilities and IoC-related support
  Beans     Bean definition and bean creation infrastructure
  Context   ApplicationContext and application-level services
  SpEL      Spring Expression Language

## 3.2 Spring AOP

AOP means **Aspect-Oriented Programming**.

It helps separate cross-cutting concerns from business logic.

Examples of cross-cutting concerns: - Logging - Security checks -
Transaction management - Performance monitoring

For example, instead of writing logging code in every business method,
an aspect can apply logging behavior to selected methods.

## 3.3 Data Access and Integration

Spring provides support for database access and transaction management.

Examples: - JDBC support - Transaction abstraction - Integration with
ORM technologies such as Hibernate

Spring does not require every application to use one specific database
or ORM technology.

## 3.4 Spring Web and Spring MVC

Spring Web provides web-related infrastructure.

Spring MVC is Spring's servlet-based web framework. It supports the
Model-View-Controller pattern and is commonly used to build web
applications and REST APIs.

------------------------------------------------------------------------

# 4. Spring Core: IoC and Dependency Injection

These are among the most important Spring interview topics.

## 4.1 What is a dependency?

A dependency is an object that another object needs to perform its work.

Example:

``` java
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine;

    public Car() {
        engine = new Engine();
    }

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

Here, `Car` depends on `Engine`.

The problem is that `Car` creates its own `Engine`. This makes it harder
to replace the engine implementation or provide a test object.

## 4.2 What is tight coupling?

Tight coupling occurs when one class is strongly dependent on a specific
implementation of another class.

``` java
class Car {
    private PetrolEngine engine = new PetrolEngine();
}
```

The `Car` class is directly tied to `PetrolEngine`. Replacing it with an
electric engine requires changing the class.

## 4.3 What is loose coupling?

Loose coupling means classes have fewer direct dependencies on specific
implementations.

An interface can help:

``` java
interface Engine {
    void start();
}

class PetrolEngine implements Engine {
    public void start() {
        System.out.println("Petrol engine started");
    }
}

class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric engine started");
    }
}

class Car {
    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
    }
}
```

Now `Car` depends on the `Engine` abstraction rather than a specific
engine class.

## 4.4 What is Inversion of Control (IoC)?

**Inversion of Control means that control over creating and managing
objects is transferred from application code to a framework or
container.**

Without IoC:

``` java
Engine engine = new PetrolEngine();
Car car = new Car(engine);
```

The application code creates and connects the objects.

With Spring IoC, Spring's container can create the objects and connect
their dependencies.

The developer describes the components and their relationships; the
container manages them.

## 4.5 What is Dependency Injection (DI)?

Dependency Injection is a technique used to implement IoC.

Instead of a class creating its dependencies, the dependencies are
supplied from outside.

Spring supports: 1. Constructor injection 2. Setter injection 3. Field
injection

### Constructor injection

``` java
@Component
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

@Component
class Car {
    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

Spring creates the `Engine` bean and passes it to the `Car` constructor.

**Recommended for required dependencies**, because: - The dependency is
clearly visible - The object can be immutable - The object cannot be
created without the required dependency - It is easy to test using
ordinary Java code

If a class has one constructor, Spring can use it without requiring
`@Autowired` on that constructor.

### Setter injection

``` java
@Component
class Car {
    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Setter injection is useful when a dependency is optional or needs to be
changed after object construction.

### Field injection

``` java
@Component
class Car {
    @Autowired
    private Engine engine;
}
```

Field injection is concise, but it hides dependencies and makes ordinary
unit testing more difficult. Constructor injection is generally
preferred for required dependencies.

## 4.6 IoC vs DI

  -----------------------------------------------------------------------
  IoC                                 DI
  ----------------------------------- -----------------------------------
  A broader design principle          A technique

  Control is transferred to a         Dependencies are supplied from
  container or framework              outside

  Can be implemented in different     Commonly used to implement IoC
  ways                                
  -----------------------------------------------------------------------

**Interview answer:** IoC is the principle of handing object creation
and management to a container. Dependency Injection is one way to
achieve that principle by supplying required objects to a class.

------------------------------------------------------------------------

# 5. Spring Beans

## 5.1 What is a Spring Bean?

A Spring Bean is an object that is created, configured, and managed by
the Spring IoC container.

A normal Java object is not automatically a Spring bean.

Example:

``` java
@Component
public class StudentService {
    public void display() {
        System.out.println("Student service");
    }
}
```

When component scanning finds this class, Spring can register and manage
it as a bean.

## 5.2 How does Spring create beans?

A simplified lifecycle:

``` text
Read configuration
       |
       v
Find bean definitions
       |
       v
Create bean instance
       |
       v
Inject dependencies
       |
       v
Run initialization callbacks
       |
       v
Bean is ready to use
       |
       v
Container shutdown
       |
       v
Run destruction callbacks
```

This is a simplified lifecycle. Actual lifecycle processing can include
bean post-processors, proxy creation, and other steps.

## 5.3 Bean naming

By default, a component class named `StudentService` is commonly
registered with the bean name `studentService`.

``` java
@Component("studentManager")
public class StudentService {
}
```

Here, the bean name is `studentManager`.

## 5.4 `@Component`, `@Service`, `@Repository`, and `@Controller`

These annotations identify classes as Spring-managed components.

  Annotation      Typical use
  --------------- --------------------------------
  `@Component`    General-purpose component
  `@Service`      Business/service layer
  `@Repository`   Data access/DAO layer
  `@Controller`   MVC controller returning views

`@Service`, `@Repository`, and `@Controller` are specialized forms of
`@Component`.

Using the appropriate annotation communicates the role of the class and
can enable additional framework behavior. For example, `@Repository`
participates in persistence exception translation when the relevant
infrastructure is configured.

------------------------------------------------------------------------

# 6. Bean Scopes

Bean scope defines how many instances Spring creates and how long those
instances are associated with a container or request.

  -----------------------------------------------------------------------
  Scope                               Meaning
  ----------------------------------- -----------------------------------
  `singleton`                         One bean instance per Spring IoC
                                      container

  `prototype`                         A new instance each time the
                                      container is asked for that bean

  `request`                           One instance per HTTP request

  `session`                           One instance per HTTP session

  `application`                       One instance per ServletContext

  `websocket`                         One instance per WebSocket session
  -----------------------------------------------------------------------

## Singleton scope

Singleton is the default scope.

``` java
@Component
@Scope("singleton")
class StudentService {
}
```

The container manages one shared instance of this bean within that
container.

**Important:** Spring singleton does not mean one object for the entire
JVM. It means one instance per bean definition per Spring container.

## Prototype scope

``` java
@Component
@Scope("prototype")
class Report {
}
```

Spring creates a new instance when the bean is requested from the
container.

A prototype bean's complete lifecycle is not managed by Spring after it
is handed to the client. In particular, Spring does not automatically
call its destruction callbacks in the same way as singleton beans.

## Interview point

A singleton bean may be used by many threads. Avoid storing
request-specific mutable state in singleton service fields unless thread
safety is handled.

------------------------------------------------------------------------

# 7. Spring Container and ApplicationContext

## 7.1 What is the Spring container?

The Spring container creates, configures, connects, and manages Spring
beans.

It reads bean configuration from sources such as: - Java configuration -
Component scanning - XML configuration (used in some legacy
applications)

## 7.2 BeanFactory

`BeanFactory` is the basic container interface that provides bean
management.

## 7.3 ApplicationContext

`ApplicationContext` is a commonly used, more feature-rich container
built on the BeanFactory infrastructure.

It supports features such as: - Bean management - Dependency injection -
Internationalization - Application events - Integration with additional
Spring infrastructure

In most modern Spring applications, developers work with an
`ApplicationContext` or a specialized application context.

## BeanFactory vs ApplicationContext

  BeanFactory                     ApplicationContext
  ------------------------------- -------------------------------------------
  Basic bean container            More feature-rich container
  Provides core bean management   Adds application-level features
  Basic infrastructure            Common choice for application development

------------------------------------------------------------------------

# 8. Configuration in Spring

Spring can be configured in different ways.

## 8.1 XML configuration

Older Spring applications may define beans in XML.

``` xml
<bean id="engine" class="com.example.Engine"/>
```

XML configuration is still supported, but annotation-based and
Java-based configuration are common in modern applications.

## 8.2 Annotation-based configuration

``` java
@Component
class Engine {
}
```

Spring can discover the class through component scanning.

## 8.3 Java-based configuration

``` java
@Configuration
class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }
}
```

-   `@Configuration` marks a class as a source of bean definitions.
-   `@Bean` tells Spring to register the object returned by the method
    as a bean.

Use `@Bean` when you need to configure an object explicitly, especially
when the class is from a third-party library and cannot be annotated
with `@Component`.

## 8.4 `@Component` vs `@Bean`

  -----------------------------------------------------------------------
  `@Component`                        `@Bean`
  ----------------------------------- -----------------------------------
  Applied to a class                  Applied to a method

  Usually discovered by component     Method is declared in a
  scanning                            configuration class or another
                                      suitable Spring-managed
                                      configuration source

  Convenient for classes you control  Useful for third-party or
                                      explicitly configured objects
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 9. Important Spring Annotations

  -----------------------------------------------------------------------
  Annotation                          Meaning
  ----------------------------------- -----------------------------------
  `@Component`                        Registers a component for scanning

  `@Service`                          Marks a service-layer component

  `@Repository`                       Marks a repository/DAO component

  `@Controller`                       Marks an MVC controller

  `@RestController`                   Controller whose handler methods
                                      write response bodies

  `@Autowired`                        Requests dependency injection

  `@Qualifier`                        Selects a particular bean when
                                      multiple candidates exist

  `@Primary`                          Gives a bean preference when
                                      multiple candidates match

  `@Configuration`                    Declares Java-based configuration

  `@Bean`                             Declares a bean-producing method

  `@ComponentScan`                    Configures component scanning

  `@Scope`                            Specifies bean scope

  `@Value`                            Injects a value from configuration
                                      or an expression

  `@Lazy`                             Requests lazy initialization for a
                                      bean or injection point, subject to
                                      context

  `@PostConstruct`                    Marks an initialization callback

  `@PreDestroy`                       Marks a destruction callback
  -----------------------------------------------------------------------

Example using `@Qualifier`:

``` java
@Component("petrolEngine")
class PetrolEngine implements Engine {
}

@Component("electricEngine")
class ElectricEngine implements Engine {
}

@Component
class Car {
    private final Engine engine;

    public Car(@Qualifier("electricEngine") Engine engine) {
        this.engine = engine;
    }
}
```

If multiple beans implement the same interface, Spring needs a way to
determine which bean should be injected. `@Qualifier` helps identify the
desired candidate.

------------------------------------------------------------------------

# 10. Spring MVC

## 10.1 What is Spring MVC?

Spring MVC is a web framework in Spring used to build web applications
and web APIs using the Model-View-Controller pattern.

MVC stands for: - **Model:** Data used by the application or presented
to the view - **View:** The presentation layer, such as a JSP or
template - **Controller:** Receives web requests and coordinates the
response

Spring MVC is based on the Servlet API and uses a central component
called `DispatcherServlet`.

## 10.2 Why use Spring MVC?

Spring MVC provides: - Request mapping - Controller methods - Model and
view support - Form handling - Data binding - Validation integration -
Exception handling - REST API support - Integration with other Spring
features

## 10.3 MVC architecture

``` text
Browser / Client
       |
       | HTTP Request
       v
DispatcherServlet
       |
       v
Controller
       |
       v
Service Layer
       |
       v
Repository / DAO
       |
       v
Database
```

The response travels back through the web application to the client. For
a traditional MVC page, a view may render the model data. For a REST
API, the controller typically returns data that is serialized into JSON
or another response format.

## 10.4 Model, View and Controller

### Model

The model represents data needed by the presentation layer.

Example:

``` java
model.addAttribute("studentName", "Ravi");
```

### View

The view displays the data. Examples include JSP, Thymeleaf templates,
or another configured view technology.

### Controller

The controller handles incoming requests and selects the next action or
response.

``` java
@Controller
public class StudentController {

    @GetMapping("/student")
    public String showStudent(Model model) {
        model.addAttribute("studentName", "Ravi");
        return "student";
    }
}
```

In a conventional view-based application, returning `"student"` usually
identifies a logical view name. A configured view resolver determines
the actual view.

------------------------------------------------------------------------

# 11. Spring MVC Request Processing Flow

This is a very common interview question.

``` text
1. Client sends HTTP request
            |
            v
2. DispatcherServlet receives request
            |
            v
3. HandlerMapping finds matching handler
            |
            v
4. HandlerAdapter invokes controller method
            |
            v
5. Controller calls service layer
            |
            v
6. Service calls repository / DAO
            |
            v
7. Data is returned to controller
            |
            v
8. Controller returns view name or response body
            |
            v
9a. ViewResolver finds view       9b. HttpMessageConverter
    and view renders model            serializes response data
            |                         |
            +------------+------------+
                         |
                         v
10. DispatcherServlet sends HTTP response
                         |
                         v
11. Client receives response
```

### Important components

**DispatcherServlet:** Front controller that coordinates request
processing.

**HandlerMapping:** Finds the handler/controller method that matches the
request.

**HandlerAdapter:** Invokes the selected handler using the appropriate
mechanism.

**ViewResolver:** Resolves a logical view name to a view implementation.

**HttpMessageConverter:** Converts request or response bodies, such as
JSON, to and from Java objects.

**Interview answer:** DispatcherServlet is the front controller in
Spring MVC. It receives requests and delegates to components such as
HandlerMapping, HandlerAdapter, and ViewResolver or message converters
to produce the response.

------------------------------------------------------------------------

# 12. Controllers, Model and Views

## 12.1 `@Controller`

Used for MVC controllers that commonly return view names.

``` java
@Controller
public class HomeController {

    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Welcome");
        return "home";
    }
}
```

The view name `home` is resolved by the configured view technology.

## 12.2 `Model`

`Model` allows a controller to add attributes that can be accessed by
the view.

``` java
model.addAttribute("message", "Hello");
```

## 12.3 `ModelAndView`

`ModelAndView` can carry both a model and a view name.

``` java
@GetMapping("/about")
public ModelAndView about() {
    ModelAndView mav = new ModelAndView("about");
    mav.addObject("message", "About page");
    return mav;
}
```

## 12.4 `@RestController`

`@RestController` is a convenience annotation that combines
`@Controller` and `@ResponseBody`.

``` java
@RestController
public class GreetingController {

    @GetMapping("/greeting")
    public String greeting() {
        return "Hello";
    }
}
```

The return value is written to the HTTP response body rather than
interpreted as a view name.

------------------------------------------------------------------------

# 13. REST APIs with Spring MVC

## What is REST?

REST is an architectural style for designing networked applications.
RESTful APIs commonly expose resources through URLs and use HTTP methods
to operate on those resources.

Example resource:

``` text
/students
/students/10
```

Common HTTP methods:

  Method   Common purpose
  -------- ----------------------------------
  GET      Read a resource
  POST     Create a resource or submit data
  PUT      Replace a resource
  PATCH    Partially update a resource
  DELETE   Delete a resource

## JSON and Java objects

A REST API often receives JSON and converts it into a Java object.

``` java
public class Student {
    private int id;
    private String name;

    public Student() {
    }

    public Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

Example controller:

``` java
@RestController
@RequestMapping("/api/students")
public class StudentRestController {

    @GetMapping("/{id}")
    public Student getStudent(@PathVariable int id) {
        return new Student(id, "Ravi");
    }

    @PostMapping
    public Student createStudent(@RequestBody Student student) {
        return student;
    }
}
```

In a typical Spring Boot web application with JSON support configured, a
message converter serializes the returned object into JSON.

## `@RequestBody`

Reads the HTTP request body and converts it into a Java object.

## `@ResponseBody`

Writes the return value of a controller method to the HTTP response
body.

## `@PathVariable`

Reads a value from the URL path.

``` java
@GetMapping("/students/{id}")
public String find(@PathVariable int id) {
    return "Student ID: " + id;
}
```

## `@RequestParam`

Reads a query parameter or form parameter.

``` java
@GetMapping("/search")
public String search(@RequestParam String name) {
    return "Searching for " + name;
}
```

Request example:

``` text
/search?name=Ravi
```

------------------------------------------------------------------------

# 14. Request Mapping and HTTP Methods

## `@RequestMapping`

Maps a URL pattern and optionally an HTTP method to a controller or
handler method.

``` java
@RequestMapping("/students")
```

It can be applied at class level or method level.

## Shortcut annotations

  Annotation         HTTP method
  ------------------ -------------
  `@GetMapping`      GET
  `@PostMapping`     POST
  `@PutMapping`      PUT
  `@PatchMapping`    PATCH
  `@DeleteMapping`   DELETE

Example:

``` java
@RestController
@RequestMapping("/api/students")
public class StudentController {

    @GetMapping
    public String list() {
        return "List students";
    }

    @PostMapping
    public String create() {
        return "Create student";
    }

    @PutMapping("/{id}")
    public String update(@PathVariable int id) {
        return "Update student " + id;
    }

    @DeleteMapping("/{id}")
    public String delete(@PathVariable int id) {
        return "Delete student " + id;
    }
}
```

------------------------------------------------------------------------

# 15. Validation and Exception Handling

## 15.1 Validation

Validation checks whether incoming data satisfies required rules.

Examples: - Name must not be blank - Email must have a valid format -
Age must be within a permitted range

A DTO can use validation annotations:

``` java
public class StudentRequest {

    @NotBlank
    private String name;

    @Email
    private String email;

    // getters and setters
}
```

Controller:

``` java
@PostMapping
public String create(@Valid @RequestBody StudentRequest request) {
    return "Student accepted";
}
```

The appropriate validation dependency must be included. Validation
behavior and exception handling depend on the Spring version and
application configuration.

## 15.2 Exception handling

An exception is an error condition that interrupts normal program
execution.

Spring MVC supports centralized exception handling.

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> handleBadInput(
            IllegalArgumentException ex) {

        return ResponseEntity
                .badRequest()
                .body(ex.getMessage());
    }
}
```

-   `@ControllerAdvice` provides shared controller-related behavior.
-   `@RestControllerAdvice` combines `@ControllerAdvice` and
    `@ResponseBody`.
-   `@ExceptionHandler` identifies a method that handles a particular
    exception type.

Centralized exception handling avoids repeating similar try-catch logic
in every controller.

------------------------------------------------------------------------

# 16. Spring Boot

## 16.1 What is Spring Boot?

Spring Boot is a framework built on top of the Spring Framework that
simplifies the development and running of Spring applications.

It helps developers create applications with less manual configuration.

## 16.2 Why Spring Boot?

Traditional Spring applications may require developers to configure many
dependencies and infrastructure components.

Spring Boot provides: - Auto-configuration - Starter dependencies -
Embedded server support - Externalized configuration -
Production-oriented features through Actuator - Convenient application
startup

## 16.3 Spring Boot does not replace Spring

Spring Boot uses Spring Framework features such as: - IoC container -
Dependency Injection - Spring beans - Spring MVC - Transaction support

Spring Boot mainly simplifies application setup and configuration.

## 16.4 What is an embedded server?

An embedded server is packaged with or launched by the application so
that the application can run without requiring a separately installed
and manually configured application server.

Spring Boot web applications commonly use an embedded servlet container
such as Tomcat.

A typical executable Spring Boot web application can be started using:

``` bash
java -jar application.jar
```

The exact packaging and server depend on the project's dependencies and
configuration.

## 16.5 What is a Spring Boot starter?

A starter is a convenient dependency descriptor that brings in a set of
related dependencies.

Example:

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

The web starter commonly brings in dependencies needed for Spring MVC
and web application development, including an embedded servlet container
for the usual executable application setup.

Other common starters include: - `spring-boot-starter-test` -
`spring-boot-starter-data-jpa` - `spring-boot-starter-validation` -
`spring-boot-starter-actuator` - `spring-boot-starter-security`

A starter does not itself implement the application's business logic. It
simplifies dependency management.

------------------------------------------------------------------------

# 17. Spring Boot Auto-Configuration

## What is auto-configuration?

Auto-configuration attempts to configure Spring application
infrastructure based on: - Classes available on the classpath - Existing
beans - Application properties - Conditions defined by Spring Boot

For example, when web-related dependencies are present, Boot can
configure web application infrastructure automatically.

Auto-configuration is not magic. It is conditional configuration
supplied by Spring Boot.

## How auto-configuration works (simplified)

``` text
Spring Boot application starts
          |
          v
@SpringBootApplication is processed
          |
          v
Component scanning finds components
          |
          v
Auto-configuration checks classpath,
existing beans and configuration
          |
          v
Suitable configuration is applied
          |
          v
Application context is refreshed
          |
          v
Application starts
```

## `@SpringBootApplication`

This is the main annotation used on a typical Spring Boot application
class.

It combines: - `@SpringBootConfiguration` - `@EnableAutoConfiguration` -
`@ComponentScan`

Example:

``` java
@SpringBootApplication
public class StudentApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudentApplication.class, args);
    }
}
```

### `@SpringBootConfiguration`

Indicates that the class provides Spring Boot configuration. It is a
specialized form of `@Configuration`.

### `@EnableAutoConfiguration`

Enables Spring Boot's auto-configuration mechanism.

### `@ComponentScan`

Scans for Spring components in the configured package and its
subpackages by default.

**Interview point:** Place the main application class in a root package
above the application's controllers, services, and repositories so
component scanning can find them naturally.

## Can auto-configuration be overridden?

Yes. Developers can define their own beans or change configuration
properties. Depending on the auto-configuration condition, Boot may back
off when a user-defined bean is present.

------------------------------------------------------------------------

# 18. Spring Boot Project Structure

A common Maven project structure:

``` text
student-management/
|
+-- src/
|   +-- main/
|   |   +-- java/
|   |   |   +-- com/example/student/
|   |   |       +-- StudentApplication.java
|   |   |       +-- controller/
|   |   |       +-- service/
|   |   |       +-- repository/
|   |   |       +-- entity/
|   |   |       +-- dto/
|   |   |
|   |   +-- resources/
|   |       +-- application.properties
|   |       +-- static/
|   |       +-- templates/
|   |
|   +-- test/
|       +-- java/
|
+-- pom.xml
```

## Common layers

### Controller layer

Receives HTTP requests and returns HTTP responses.

### Service layer

Contains business logic and coordinates operations.

### Repository / DAO layer

Communicates with the database.

### Entity / Model

Represents data, often mapped to database tables in persistence
applications.

### DTO (Data Transfer Object)

Carries data between layers or across API boundaries. DTOs help avoid
exposing internal entity structures directly.

## Typical application flow

``` text
Client
  |
  v
Controller
  |
  v
Service
  |
  v
Repository / DAO
  |
  v
Database
```

A controller should generally not contain complex business logic or
direct database code. Keep responsibilities separated.

------------------------------------------------------------------------

# 19. Configuration Properties and Profiles

## 19.1 `application.properties`

Spring Boot commonly reads configuration from `application.properties`
or `application.yml`.

Example:

``` properties
server.port=8081
spring.application.name=student-management
```

The application will normally use port 8081 if no other setting
overrides it.

## 19.2 `@Value`

``` java
@Value("${server.port}")
private int port;
```

It can inject a configured property into a bean.

## 19.3 `@ConfigurationProperties`

For a group of related configuration values, `@ConfigurationProperties`
is often more maintainable than many individual `@Value` fields.

Example:

``` properties
school.name=Codegnan
school.city=Hyderabad
```

Configuration class:

``` java
@ConfigurationProperties(prefix = "school")
public class SchoolProperties {
    private String name;
    private String city;

    // getters and setters
}
```

The class must be registered for configuration properties binding, for
example through configuration-properties scanning or
`@EnableConfigurationProperties`.

## 19.4 Profiles

Profiles allow different configuration for different environments.

Common environments: - Development - Testing - Production

Example files:

``` text
application.properties
application-dev.properties
application-test.properties
application-prod.properties
```

Activate a profile using configuration such as:

``` properties
spring.profiles.active=dev
```

Avoid committing real production passwords or API keys into source
control. Use environment variables or an appropriate secret-management
system.

------------------------------------------------------------------------

# 20. Spring Core vs Spring MVC vs Spring Boot

  -----------------------------------------------------------------------
  Spring Core             Spring MVC              Spring Boot
  ----------------------- ----------------------- -----------------------
  Fundamental Spring      Web framework           Application setup and
  container features                              configuration
                                                  simplification

  IoC and DI              Handles HTTP requests   Auto-configuration

  Bean management         Controllers and request Starter dependencies
                          mapping                 

  ApplicationContext      DispatcherServlet       Embedded server support

  Used in many kinds of   Used for servlet-based  Can simplify web,
  Spring applications     web apps and APIs       batch, and other Spring
                                                  applications
  -----------------------------------------------------------------------

## How they work together

A typical Spring Boot web application uses:

-   Spring Core to create and manage beans
-   Spring MVC to handle HTTP requests
-   Spring Boot to configure and launch the application conveniently

------------------------------------------------------------------------

# 21. Typical Spring Boot REST Application

This small example shows the relationship between controller, service,
and Spring-managed beans.

## Service

``` java
@Service
public class GreetingService {

    public String greet(String name) {
        return "Hello, " + name;
    }
}
```

## Controller

``` java
@RestController
@RequestMapping("/api")
public class GreetingController {

    private final GreetingService greetingService;

    public GreetingController(GreetingService greetingService) {
        this.greetingService = greetingService;
    }

    @GetMapping("/greet")
    public String greet(@RequestParam(defaultValue = "Student") String name) {
        return greetingService.greet(name);
    }
}
```

## Application class

``` java
@SpringBootApplication
public class GreetingApplication {

    public static void main(String[] args) {
        SpringApplication.run(GreetingApplication.class, args);
    }
}
```

## What happens?

1.  Spring Boot starts the application.
2.  Component scanning finds the controller and service.
3.  Spring creates the `GreetingService` bean.
4.  Spring creates the controller and injects the service through its
    constructor.
5.  A client sends `GET /api/greet?name=Ravi`.
6.  Spring MVC maps the request to the controller method.
7.  The controller calls the service.
8.  The returned string is written to the HTTP response body.

This example demonstrates Spring Core, Spring MVC, and Spring Boot
working together.

------------------------------------------------------------------------

# 22. Common Interview Questions and Answers

## Spring Core

### Q1. What is Spring?

Spring is a Java framework that provides infrastructure for developing
applications. Its core features include IoC, Dependency Injection, bean
management, and integration with web and data technologies.

### Q2. What is IoC?

IoC is a design principle in which control over creating and managing
objects is transferred from application code to a container or
framework.

### Q3. What is Dependency Injection?

Dependency Injection is a technique in which an object's dependencies
are supplied from outside rather than created directly by that object.

### Q4. What is a Spring Bean?

A Spring bean is an object managed by the Spring IoC container.

### Q5. What is the default bean scope?

The default scope is `singleton`, which means one instance per bean
definition per Spring container.

### Q6. What is ApplicationContext?

ApplicationContext is a feature-rich Spring container that manages beans
and provides application-level features such as events and
internationalization.

### Q7. What is the difference between `@Component` and `@Bean`?

`@Component` is applied to a class for component scanning. `@Bean` is
applied to a method that explicitly creates and registers a bean.

### Q8. What is constructor injection?

Constructor injection supplies required dependencies through a class
constructor. It makes dependencies explicit and supports immutable
design.

### Q9. What is `@Autowired`?

`@Autowired` tells Spring to resolve and inject a suitable dependency at
the annotated injection point.

### Q10. What happens when multiple beans of the same type exist?

Spring may not know which candidate to inject. Use `@Qualifier` to
select a bean or `@Primary` to designate a preferred candidate.

## Spring MVC

### Q11. What is Spring MVC?

Spring MVC is a servlet-based Spring web framework that follows the
Model-View-Controller pattern and supports web applications and REST
APIs.

### Q12. What is DispatcherServlet?

DispatcherServlet is Spring MVC's front controller. It receives incoming
requests and coordinates request mapping, handler invocation, and
response processing.

### Q13. What is a front controller?

A front controller is a central entry point that receives requests and
delegates them to appropriate handlers.

### Q14. What is the role of HandlerMapping?

HandlerMapping identifies the handler that should process a request.

### Q15. What is the role of ViewResolver?

ViewResolver maps a logical view name returned by a controller to an
actual view implementation.

### Q16. What is the difference between `@Controller` and `@RestController`?

`@Controller` is commonly used for view-based MVC. `@RestController`
writes handler return values directly to the response body by default.

### Q17. What is `@RequestBody`?

It binds the HTTP request body to a Java object using an appropriate
message converter.

### Q18. What is `@PathVariable`?

It binds a value from a URL path variable to a method parameter.

### Q19. What is `@RequestParam`?

It binds a request parameter, commonly a query parameter or form
parameter, to a method argument.

### Q20. What is the difference between Model and ModelAndView?

`Model` carries model attributes. `ModelAndView` carries both model
attributes and a view name or view reference.

## Spring Boot

### Q21. What is Spring Boot?

Spring Boot simplifies Spring application development by providing
auto-configuration, starter dependencies, convenient application
startup, and other defaults.

### Q22. What is auto-configuration?

Auto-configuration conditionally configures application infrastructure
based on classpath contents, existing beans, and configuration
properties.

### Q23. What is `@SpringBootApplication`?

It combines `@SpringBootConfiguration`, `@EnableAutoConfiguration`, and
`@ComponentScan`.

### Q24. What is a starter dependency?

A starter is a dependency descriptor that brings in a set of related
dependencies for a particular type of application or feature.

### Q25. What is an embedded server?

An embedded server is included in or launched by the application,
allowing it to run without a separately managed external server in the
usual executable setup.

### Q26. Can Spring Boot be used without Spring MVC?

Yes. Spring Boot can be used for applications that do not use Spring
MVC, such as command-line, batch, or other types of applications. It can
also support reactive web applications through other Spring
technologies.

### Q27. Does Spring Boot eliminate configuration?

No. It reduces manual configuration by providing sensible defaults and
conditional auto-configuration. Developers can still configure or
override application behavior.

### Q28. What is the difference between Spring and Spring Boot?

Spring Framework provides core application features and modules. Spring
Boot builds on Spring and simplifies configuration, dependency setup,
and application execution.

------------------------------------------------------------------------

# 23. Important Differences for Interviews

## IoC vs DI

-   IoC is the principle of transferring control to a container.
-   DI is a technique of supplying dependencies from outside.

## BeanFactory vs ApplicationContext

-   BeanFactory provides basic bean management.
-   ApplicationContext provides bean management plus additional
    application-level features.

## Singleton vs Prototype

-   Singleton: one managed instance per bean definition per container.
-   Prototype: a new instance each time the container is asked for that
    bean.

## `@Controller` vs `@RestController`

-   `@Controller`: commonly returns a view name.
-   `@RestController`: writes return values to the response body.

## `@RequestParam` vs `@PathVariable`

-   `@RequestParam`: reads request parameters, such as `?name=Ravi`.
-   `@PathVariable`: reads values embedded in the URL path, such as
    `/students/10`.

## `@Component` vs `@Service`

-   Both can register Spring-managed components.
-   `@Service` communicates that the class belongs to the
    business/service layer.

## Spring vs Spring Boot

-   Spring provides the core framework and modules.
-   Spring Boot simplifies building and running Spring applications.

## Traditional Spring MVC deployment vs Spring Boot executable application

-   A traditional application may be packaged as a WAR and deployed to
    an externally managed servlet container.
-   A typical Spring Boot web application can be packaged as an
    executable JAR with an embedded servlet container.
-   Spring Boot can also be packaged as a WAR when required.

------------------------------------------------------------------------

# 24. Fresher Interview Revision Checklist

Before an interview, make sure you can explain these topics in your own
words:

## Spring Core

-   [ ] What is Spring?
-   [ ] What is IoC?
-   [ ] What is Dependency Injection?
-   [ ] What is tight coupling and loose coupling?
-   [ ] What is a Spring Bean?
-   [ ] What is the Spring IoC container?
-   [ ] What is ApplicationContext?
-   [ ] What are bean scopes?
-   [ ] What is constructor injection?
-   [ ] Difference between `@Component`, `@Service`, and `@Repository`
-   [ ] Difference between `@Component` and `@Bean`
-   [ ] What are `@Qualifier` and `@Primary`?

## Spring MVC

-   [ ] What is MVC?
-   [ ] What is DispatcherServlet?
-   [ ] Explain Spring MVC request processing.
-   [ ] What are HandlerMapping and HandlerAdapter?
-   [ ] What is ViewResolver?
-   [ ] Difference between `@Controller` and `@RestController`
-   [ ] Difference between `@RequestParam` and `@PathVariable`
-   [ ] What is `@RequestBody`?
-   [ ] What is `@ResponseBody`?
-   [ ] How are exceptions handled globally?

## Spring Boot

-   [ ] What is Spring Boot?
-   [ ] Why is Spring Boot used?
-   [ ] What is auto-configuration?
-   [ ] What is `@SpringBootApplication`?
-   [ ] What are starter dependencies?
-   [ ] What is an embedded server?
-   [ ] What is `application.properties`?
-   [ ] What are profiles?
-   [ ] What is the purpose of the service layer?
-   [ ] Explain the flow from Controller to Service to Repository.

------------------------------------------------------------------------

# 25. Glossary

  -----------------------------------------------------------------------
  Term                                Simple meaning
  ----------------------------------- -----------------------------------
  Framework                           Reusable structure that helps build
                                      applications

  IoC                                 Giving object management control to
                                      a container

  DI                                  Supplying an object's dependencies
                                      from outside

  Bean                                Object managed by Spring

  Container                           Component that creates and manages
                                      beans

  ApplicationContext                  Feature-rich Spring container

  Scope                               Defines bean instance sharing and
                                      lifecycle

  MVC                                 Model-View-Controller design
                                      pattern

  DispatcherServlet                   Spring MVC front controller

  Controller                          Component that handles web requests

  REST API                            Web interface that exposes
                                      resources through HTTP

  Auto-configuration                  Conditional configuration provided
                                      by Spring Boot

  Starter                             Convenient group of related
                                      dependencies

  Embedded server                     Server launched as part of the
                                      application

  DTO                                 Object used to transfer data

  Repository / DAO                    Component responsible for data
                                      access

  Service                             Component containing business logic
  -----------------------------------------------------------------------

------------------------------------------------------------------------
A strong fresher answer is usually a clear explanation followed by a
small practical example.
