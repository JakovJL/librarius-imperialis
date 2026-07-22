# Design Patterns for AQA

## Table of Contents

- [[#Why Design Patterns Matter]]
- [[#Pattern Categories]]
	- [[#Creational Patterns]]
	- [[#Structural Patterns]]
	- [[#Behavioral Patterns]]
- [[#Factory Pattern]]
	- [[#Simple Factory]]
	- [[#Factory Method]]
- [[#Builder Pattern]]
	- [[#Test Data Builder]]
	- [[#Request Specification Builder]]
- [[#Strategy Pattern]]
	- [[#Authentication Strategy]]
- [[#Choosing a Pattern]]
- [[#Combining Patterns in an AQA Framework]]
- [[#Common Mistakes]]
- [[#Interview Questions]]
	- [[#Beginner Questions]]
	- [[#Intermediate Questions]]
	- [[#Advanced Questions]]
	- [[#Code Questions]]

**Related notes:** [[AQA Java eng]]

> [!caution] Not in the video course
> Zaur's courses end at Java Core — design patterns for AQA are not covered. This is the AQA layer: learn from this note and practice, and apply a pattern only when it solves a real design problem.

---

## Why Design Patterns Matter

A **design pattern** is a reusable design idea for a common software problem. It describes how classes and objects can work together; it is not finished code that must be copied exactly.

**Goal:** make code easier to extend, test, and maintain by using solutions that other developers recognize.

In an AQA framework, patterns can help with:

- creating browser drivers or API clients
- building test data with clear defaults
- changing behavior at runtime
- separating test scenarios from technical setup
- reducing large `if` and `switch` blocks

Patterns are useful when change or duplication already exists. A direct constructor, one small method, or a simple conditional is often enough for a small problem.

> [!danger] Solve the Problem First
> Do not add a pattern only because it is popular. A pattern should remove a real problem, not add extra classes without value.

---

## Pattern Categories

The classic categories describe what kind of design problem a pattern solves.

### Creational Patterns

**Definition:** creational patterns control how objects are created.

**Goal:** hide complex construction and reduce direct dependency on concrete classes.

Common examples:

- Factory Method
- Abstract Factory
- Builder
- Prototype
- Singleton

In AQA, Factory and Builder are common for drivers, clients, configuration, and test data.

### Structural Patterns

**Definition:** structural patterns organize classes and objects into larger structures.

**Goal:** connect components without making them tightly dependent on one another.

Common examples:

- Adapter
- Decorator
- Facade
- Proxy
- Composite

In AQA, an Adapter can hide differences between third-party tools, while a Facade can provide a simpler interface to a complex subsystem.

### Behavioral Patterns

**Definition:** behavioral patterns organize communication and responsibility between objects.

**Goal:** make behavior replaceable and keep decision logic out of large conditionals.

Common examples:

- Strategy
- Observer
- Command
- Template Method
- Chain of Responsibility

In AQA, Strategy can select authentication, retry, waiting, or data-generation behavior at runtime.

> [!tip] Quick Comparison
> | Category | Main question | Pattern in this note | AQA example |
> |---|---|---|---|
> | Creational | How is an object created? | Factory, Builder | Create a driver or test user |
> | Structural | How are objects connected? | Not covered in depth here | Adapt a reporting library |
> | Behavioral | How can behavior change? | Strategy | Select an authentication method |

---

## Factory Pattern

**Definition:** a Factory centralizes object creation and returns an object through a common type.

**Goal:** remove direct construction of concrete implementations from tests and other client code.

**How it works:** the caller asks for a type or option. The factory chooses the concrete class, creates it, and returns an interface or base type.

**Advantages:**

- hides construction details
- keeps tests independent from concrete implementations
- places creation logic in one location
- makes new implementations easier to add

**Disadvantages:**

- adds another abstraction and more classes
- can become a large conditional if it handles too many responsibilities
- does not remove the need to configure and close created resources

### Simple Factory

A Simple Factory is a class or method that chooses an implementation, often with `switch`. It is commonly called a factory, but it is not the GoF Factory Method pattern.

Example for browser creation:

```java
public enum Browser {
    CHROME,
    FIREFOX,
    EDGE
}
```

```java
public final class BrowserFactory {

    private BrowserFactory() {
    }

    public static WebDriver create(Browser browser) {
        return switch (browser) {
            case CHROME -> new ChromeDriver();
            case FIREFOX -> new FirefoxDriver();
            case EDGE -> new EdgeDriver();
        };
    }
}
```

```java
WebDriver driver = BrowserFactory.create(Browser.CHROME);

try {
    driver.get("https://example.com");
} finally {
    driver.quit();
}
```

The test depends on `WebDriver`, not directly on one concrete driver class. The factory chooses the implementation.

### Factory Method

Factory Method defines a creation method in a base type and lets subclasses choose the concrete product.

```java
public abstract class DriverProvider {

    public abstract WebDriver createDriver();
}
```

```java
public final class ChromeDriverProvider extends DriverProvider {

    @Override
    public WebDriver createDriver() {
        return new ChromeDriver();
    }
}
```

```java
public final class FirefoxDriverProvider extends DriverProvider {

    @Override
    public WebDriver createDriver() {
        return new FirefoxDriver();
    }
}
```

**When to use:** use Factory Method when creation should vary through inheritance or each provider needs its own construction steps. For a small browser choice, a Simple Factory is usually easier.

> [!tip] Simple Factory vs Factory Method
> | Point | Simple Factory | Factory Method |
> |---|---|---|
> | Main mechanism | One method selects an implementation | Subclasses override a creation method |
> | GoF pattern | No | Yes |
> | Number of classes | Usually smaller | Usually larger |
> | Good AQA use | Small driver or client selection | Providers with different setup logic |

---

## Builder Pattern

**Definition:** Builder creates a complex object step by step and produces the final object with `build()`.

**Goal:** make object construction readable when there are many optional values, defaults, or validation rules.

**How it works:** the builder stores temporary values. Fluent methods change selected values and return the same builder. `build()` validates the state and creates the final object.

**Advantages:**

- makes test data readable
- supports useful defaults
- avoids constructors with many parameters
- allows reusable object variations
- can validate data before object creation

**Disadvantages:**

- adds extra code for simple objects
- bad defaults can hide important test data
- a mutable builder should not be shared between parallel tests

### Test Data Builder

Assume a test needs user data:

```java
public record UserData(
        String name,
        String email,
        String role,
        boolean active
) {
}
```

A builder gives valid defaults and lets each test change only important fields:

```java
public final class UserDataBuilder {
    private String name = "Test User";
    private String email = "test.user@example.com";
    private String role = "USER";
    private boolean active = true;

    public UserDataBuilder withName(String name) {
        this.name = name;
        return this;
    }

    public UserDataBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    public UserDataBuilder withRole(String role) {
        this.role = role;
        return this;
    }

    public UserDataBuilder inactive() {
        this.active = false;
        return this;
    }

    public UserData build() {
        if (email == null || !email.contains("@")) {
            throw new IllegalStateException("A valid email is required");
        }

        return new UserData(name, email, role, active);
    }
}
```

```java
UserData admin = new UserDataBuilder()
    .withName("Ana")
    .withEmail("ana@example.com")
    .withRole("ADMIN")
    .build();
```

The test clearly shows what makes this user special. Other valid fields come from defaults.

### Request Specification Builder

REST Assured provides a real Builder example:

```java
RequestSpecification specification = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setBasePath("/v1")
    .setContentType(ContentType.JSON)
    .addHeader("Accept", "application/json")
    .build();
```

Each method configures one part. `build()` returns the final `RequestSpecification`.

> [!tip] Builder vs Constructor
> | Point | Constructor | Builder |
> |---|---|---|
> | Best for | A few required values | Many optional values or defaults |
> | Readability | Parameter meaning may be unclear | Fluent method names explain values |
> | Validation | Usually inside the constructor | Often inside `build()` |
> | Extra code | Low | Higher |

---

## Strategy Pattern

**Definition:** Strategy puts replaceable behavior behind a common interface.

**Goal:** choose or change an algorithm at runtime without changing the class that uses it.

**How it works:** a context receives a strategy object and delegates one responsibility to it. Different strategies implement the same interface in different ways.

**Advantages:**

- replaces large conditionals with polymorphism
- supports runtime behavior selection
- keeps each algorithm in a separate class
- makes behavior easy to test in isolation

**Disadvantages:**

- creates more small classes
- the caller must know which strategy to choose
- it is unnecessary when there is only one stable behavior

### Authentication Strategy

An API client may support different authentication methods.

```java
public interface AuthStrategy {
    RequestSpecification apply(RequestSpecification request);
}
```

```java
public final class BearerTokenStrategy implements AuthStrategy {
    private final String token;

    public BearerTokenStrategy(String token) {
        this.token = token;
    }

    @Override
    public RequestSpecification apply(RequestSpecification request) {
        return request.auth().oauth2(token);
    }
}
```

```java
public final class BasicAuthStrategy implements AuthStrategy {
    private final String username;
    private final String password;

    public BasicAuthStrategy(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public RequestSpecification apply(RequestSpecification request) {
        return request.auth().preemptive().basic(username, password);
    }
}
```

The API client delegates authentication to the selected strategy:

```java
public final class ApiClient {
    private final String baseUri;
    private final AuthStrategy authStrategy;

    public ApiClient(String baseUri, AuthStrategy authStrategy) {
        this.baseUri = baseUri;
        this.authStrategy = authStrategy;
    }

    public Response get(String path) {
        RequestSpecification request = given().baseUri(baseUri);

        return authStrategy.apply(request)
            .when()
            .get(path);
    }
}
```

```java
AuthStrategy auth = new BearerTokenStrategy(token);
ApiClient client = new ApiClient("https://api.example.com", auth);

Response response = client.get("/profile");
```

The `ApiClient` does not need `if (authType == ...)`. A new authentication method can be added as another strategy.

> [!tip] Strategy vs Conditional
> | Point | Conditional | Strategy |
> |---|---|---|
> | Good for | One small stable choice | Several behaviors that change or grow |
> | Extension | Modify the existing conditional | Add another implementation |
> | Testing | Branches are tested together | Each strategy is tested separately |
> | Cost | Very low | More classes and interfaces |

---

## Choosing a Pattern

Choose a pattern from the problem, not from the class name you want to create.

| Problem | Suitable pattern | AQA example | Simpler option |
|---|---|---|---|
| Client code should not know the concrete class | Factory | Create `ChromeDriver` or `FirefoxDriver` | Direct constructor for one implementation |
| Object has many optional fields and useful defaults | Builder | Create test users or request specifications | Constructor for a small value object |
| One behavior must be replaceable at runtime | Strategy | Select authentication or retry logic | Small `if` for two stable cases |

Ask these questions first:

1. What change is difficult today?
2. What duplication will the pattern remove?
3. Can a smaller method or class solve the same problem?
4. Will another developer understand the abstraction without a long explanation?
5. Can each component be tested independently?

---

## Combining Patterns in an AQA Framework

Patterns solve different problems and can work together:

1. A **Factory** selects and creates the required `WebDriver`.
2. A **Builder** creates test data for the scenario.
3. A **Strategy** selects authentication or another replaceable behavior.
4. The test uses these objects but does not contain their construction details.

Example framework mapping:

```text
src/test/java
├── driver
│   ├── Browser.java
│   └── BrowserFactory.java
├── testdata
│   └── UserDataBuilder.java
├── api
│   ├── ApiClient.java
│   └── auth
│       ├── AuthStrategy.java
│       ├── BasicAuthStrategy.java
│       └── BearerTokenStrategy.java
└── tests
    └── UserAccessTest.java
```

This separation supports the framework principles from [[20 Automation Framework Architecture]] and the design principles from [[23 Software Engineering Practices for AQA]].

> [!info] Patterns Have Different Responsibilities
> Factory answers “which object should be created?”, Builder answers “how should a complex object be assembled?”, and Strategy answers “which behavior should be used?”.

---

## Common Mistakes

| Mistake | Why it is a problem | Better approach |
|---|---|---|
| Adding a pattern before a real need appears | The code becomes harder without removing a problem | Start simple and refactor when change or duplication appears |
| Calling every creation helper a Factory Method | It confuses Simple Factory with the GoF pattern | Name the exact pattern used |
| Putting setup, logging, waits, and cleanup into one factory | The factory gets too many responsibilities | Keep the factory focused on creation |
| Using a Builder for an object with two required fields | Extra code gives little value | Use a constructor or factory method |
| Hiding important scenario values in Builder defaults | The test becomes difficult to understand | Show values that define the scenario |
| Sharing one mutable Builder between tests | Parallel tests can affect one another | Create a new builder per object or test |
| Creating Strategy classes for one unchanging algorithm | Many classes add no flexibility | Use one direct implementation |
| Keeping a large `switch` inside the Strategy context | Behavior is still not separated | Move each algorithm into its strategy class |
| Storing one global static `WebDriver` | Tests share browser state and break in parallel runs | Use driver instances with test or thread scope |
| Combining patterns without clear boundaries | Responsibilities become hidden across many layers | Give each pattern one specific problem |

---

## Interview Questions

### Beginner Questions

**1. What is a design pattern?**

A design pattern is a reusable design idea for a common software problem. It is not finished code that must always be copied exactly.

**2. What are the three main pattern categories?**

Creational patterns control object creation, structural patterns connect objects, and behavioral patterns organize behavior and communication.

**3. What problem does Factory solve?**

Factory centralizes object creation and hides concrete implementations from client code.

**4. What problem does Builder solve?**

Builder creates complex objects step by step, especially when they have optional values or useful defaults.

**5. What problem does Strategy solve?**

Strategy makes behavior replaceable through a common interface and allows it to be selected at runtime.

### Intermediate Questions

**1. Why should `BrowserFactory` return `WebDriver` instead of `ChromeDriver`?**

Returning the interface keeps client code independent from one browser implementation.

**2. What is the difference between Simple Factory and Factory Method?**

Simple Factory usually selects an implementation in one method. Factory Method lets subclasses override a creation method.

**3. When is Builder better than a constructor?**

Builder is better when an object has many optional values, defaults, or validation rules. A constructor is simpler for a few required values.

**4. How does Strategy support the Open/Closed Principle?**

A new behavior can be added as another strategy implementation without changing the context class.

**5. Can Factory, Builder, and Strategy be used together?**

Yes. They solve different problems: Factory creates an implementation, Builder assembles data, and Strategy supplies replaceable behavior.

### Advanced Questions

**1. Why is a large Factory dangerous?**

It may collect unrelated setup, configuration, logging, and lifecycle logic. This breaks the Single Responsibility Principle and makes changes risky.

**2. Why can Builder defaults make tests weaker?**

Important scenario data may become invisible. A reader may not know which values affect the result.

**3. When is a conditional better than Strategy?**

A small stable conditional is better when there are only one or two simple cases and no expected growth.

**4. Why is one static `WebDriver` unsafe for parallel tests?**

Tests share mutable browser state. One test can navigate, close, or change the same driver while another test uses it.

**5. How do you decide whether to introduce a pattern?**

Identify a real change or duplication problem, compare the pattern with a simpler solution, and check whether the abstraction makes the code easier to understand and test.

### Code Questions

**1. Which implementation is returned?**

```java
WebDriver driver = BrowserFactory.create(Browser.FIREFOX);
```

**Answer:** The Simple Factory returns a `FirefoxDriver` object through the `WebDriver` interface.

**2. What values does `admin` contain?**

```java
UserData admin = new UserDataBuilder()
    .withName("Ana")
    .withEmail("ana@example.com")
    .withRole("ADMIN")
    .build();
```

**Answer:** It contains name `Ana`, email `ana@example.com`, role `ADMIN`, and the default `active` value `true`.

**3. What changes when `BasicAuthStrategy` is replaced with `BearerTokenStrategy`?**

```java
AuthStrategy auth = new BearerTokenStrategy(token);
ApiClient client = new ApiClient(baseUri, auth);
```

**Answer:** The authentication behavior changes, but `ApiClient` and its request methods do not need to change.

**4. Why is this design unsafe for parallel tests?**

```java
public final class DriverStore {
    public static WebDriver driver;
}
```

**Answer:** All tests share one mutable driver. Parallel tests can overwrite, navigate, or close the same browser session. Use a driver scoped to one test or thread.
