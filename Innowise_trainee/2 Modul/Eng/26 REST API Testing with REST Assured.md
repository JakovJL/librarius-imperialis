# REST API Testing with REST Assured

## Table of Contents

- [[#Why REST Assured Matters]]
- [[#Project Setup]]
	- [[#Maven]]
	- [[#Gradle]]
	- [[#Static Imports]]
- [[#Given When Then Structure]]
	- [[#Given]]
	- [[#When]]
	- [[#Then]]
- [[#Sending Requests]]
	- [[#HTTP Methods]]
	- [[#Request Data Methods]]
	- [[#Query and Path Parameters]]
	- [[#Headers Cookies and Content Types]]
	- [[#Request Body]]
- [[#Validating Responses]]
	- [[#Status Headers and Content Type]]
	- [[#Response Body and Hamcrest Matchers]]
	- [[#JSON Path]]
	- [[#Extracting Response Data]]
	- [[#Response Time]]
- [[#Authentication]]
	- [[#Preemptive Basic Authentication]]
	- [[#Bearer Token]]
- [[#Serialization and Deserialization]]
	- [[#Serialization]]
	- [[#Deserialization]]
- [[#Request and Response Specifications]]
	- [[#RequestSpecification]]
	- [[#ResponseSpecification]]
- [[#Logging and Failure Analysis]]
- [[#JSON Schema Validation]]
- [[#Framework Structure]]
- [[#Common Mistakes]]
- [[#Interview Questions]]
	- [[#Beginner Questions]]
	- [[#Intermediate Questions]]
	- [[#Advanced Questions]]
	- [[#Code Questions]]

**Related notes:** [[AQA Java eng]]

> [!caution] Not in the video course
> Zaur's courses end at Java Core — REST Assured is not covered. This is the AQA layer: learn from this note and practice, and use the official documentation when the API changes.

---

## Why REST Assured Matters

**REST Assured** is a Java DSL for testing HTTP APIs. It sends requests, reads responses, and checks the result in automated tests.

**Goal:** make API tests readable while keeping full control over the HTTP request and response.

REST Assured supports common HTTP methods such as `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`.

It can verify:

- status codes
- headers and cookies
- JSON and XML bodies
- response time
- authentication behavior
- JSON Schema
- business rules and side effects

> [!info] Library vs Test Framework
> REST Assured is an HTTP testing library. JUnit 5 or TestNG discovers and runs tests, while REST Assured sends requests and validates responses.

---

## Project Setup

This module uses Java 21, so it can use REST Assured 6.x. REST Assured 6 requires Java 17 or newer.

### Maven

Add REST Assured with `test` scope:

```xml
<properties>
    <rest-assured.version>6.0.1</rest-assured.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>${rest-assured.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Place the REST Assured dependency before JUnit when both libraries bring Hamcrest versions into the project.

### Gradle

```groovy
dependencies {
    testImplementation 'io.rest-assured:rest-assured:6.0.1'
}
```

### Static Imports

Static imports make the DSL shorter:

```java
import static io.restassured.RestAssured.*;
import static io.restassured.matcher.RestAssuredMatchers.*;
import static org.hamcrest.Matchers.*;
```

The main `rest-assured` dependency already includes REST Assured `JsonPath` and `XmlPath` modules.

---

## Given When Then Structure

The recommended REST Assured style follows a BDD-like `given–when–then` structure.

```java
given()
    .baseUri("https://api.example.com")
    .header("Accept", "application/json")
.when()
    .get("/users/42")
.then()
    .statusCode(200)
    .body("id", equalTo(42));
```

### Given

**Definition:** `given()` creates a request specification.

Use it to prepare:

- base URI and base path
- parameters
- headers and cookies
- authentication
- content type
- request body
- logging and filters

### When

**Definition:** `when()` separates request preparation from execution.

The HTTP method after `when()` sends the request:

```java
when().get("/users");
```

### Then

**Definition:** `then()` creates a validatable response.

Use it to assert status, headers, body, and response time:

```java
then()
    .statusCode(200)
    .contentType(ContentType.JSON);
```

> [!tip] Arrange Act Assert and Given When Then
> `given` prepares test data, `when` performs the request, and `then` checks the result. This is close to Arrange–Act–Assert from unit testing.

---

## Sending Requests

### HTTP Methods

| Method | Typical purpose | REST Assured call |
|---|---|---|
| `GET` | Read a resource | `get("/users/42")` |
| `POST` | Create a resource or start an operation | `post("/users")` |
| `PUT` | Replace a resource | `put("/users/42")` |
| `PATCH` | Change part of a resource | `patch("/users/42")` |
| `DELETE` | Delete a resource | `delete("/users/42")` |
| `HEAD` | Read headers without a response body | `head("/users/42")` |
| `OPTIONS` | Read supported communication options | `options("/users")` |
| Custom method | Send another HTTP verb | `request("CONNECT", "/proxy")` |

### Request Data Methods

| Method | Purpose | Example |
|---|---|---|
| `baseUri()` | Sets scheme, host, and optional port | `baseUri("https://api.example.com")` |
| `basePath()` | Sets a common path prefix | `basePath("/v1")` |
| `queryParam()` | Adds a query parameter | `queryParam("page", 2)` |
| `pathParam()` | Replaces a path placeholder | `pathParam("id", 42)` |
| `formParam()` | Adds an HTML form parameter | `formParam("name", "Sam")` |
| `header()` / `headers()` | Adds one or more headers | `header("X-Trace-Id", traceId)` |
| `cookie()` / `cookies()` | Adds one or more cookies | `cookie("session", sessionId)` |
| `contentType()` | Sets the request `Content-Type` | `contentType(ContentType.JSON)` |
| `accept()` | Sets the expected response media type | `accept(ContentType.JSON)` |
| `body()` | Adds the request body | `body(json)` |
| `multiPart()` | Adds multipart form data or a file | `multiPart("file", file)` |
| `spec()` | Reuses a request specification | `spec(requestSpec)` |

### Query and Path Parameters

A **path parameter** is part of the URL path that points to a specific resource. Without it, the resource does not exist.

```text
GET /users/42          → user with id = 42
GET /orders/15/items/3 → item 3 in order 15
```

A **query parameter** comes after `?` and refines or filters the returned representation. The resource stays the same; only the view changes.

```text
GET /users?role=admin&active=true → all users, but only admin and active
GET /products?sort=price&page=2   → products, page 2, sorted by price
```

```java
given()
    .baseUri("https://api.example.com")
    .pathParam("id", 42)
    .queryParam("include", "roles")
.when()
    .get("/users/{id}")
.then()
    .statusCode(200);
```

The final URL:

```text
https://api.example.com/users/42?include=roles
```

> [!tip] Path vs Query Parameter
> Path = "which resource to pick". Query = "how to show it" (filter, sort, pagination).

### Headers Cookies and Content Types

```java
given()
    .header("X-Trace-Id", "test-123")
    .cookie("locale", "en")
    .contentType(ContentType.JSON)
    .accept(ContentType.JSON)
.when()
    .get("/profile");
```

- `Content-Type` describes the format of the request body.
- `Accept` describes the response formats the client can accept.
- Cookies often keep session or preference data.
- Custom headers may carry trace IDs, API versions, or feature flags.

### Request Body

A request body can be a `String`, `Map`, Java object, byte array, or file.

```java
String body = """
        {
          "name": "Sam",
          "job": "QA Engineer"
        }
        """;

given()
    .baseUri("https://api.example.com")
    .contentType(ContentType.JSON)
    .body(body)
.when()
    .post("/users")
.then()
    .statusCode(201);
```

> [!caution] Set the Content Type
> A JSON-looking string is not enough. Set `Content-Type: application/json` when the server expects JSON.

---

## Validating Responses

A useful API test checks more than one transport detail. It should validate the contract and the business result that matters to the scenario.

### Status Headers and Content Type

```java
given()
    .baseUri("https://api.example.com")
.when()
    .get("/users/42")
.then()
    .statusCode(200)
    .statusLine(containsString("OK"))
    .header("Cache-Control", containsString("max-age"))
    .contentType(ContentType.JSON);
```

| Method | What it checks |
|---|---|
| `statusCode()` | Numeric HTTP status code |
| `statusLine()` | Full status line |
| `header()` / `headers()` | Response headers |
| `cookie()` / `cookies()` | Response cookies |
| `contentType()` | Response media type |

### Response Body and Hamcrest Matchers

REST Assured uses Hamcrest matchers for readable assertions.

```java
when()
    .get("/users/42")
.then()
    .body("id", equalTo(42))
    .body("name", not(isEmptyOrNullString()))
    .body("roles", hasItem("admin"))
    .body("roles", hasSize(greaterThan(0)));
```

| Matcher | Purpose |
|---|---|
| `equalTo(value)` / `is(value)` | Exact equality |
| `not(matcher)` | Negates another matcher |
| `containsString(text)` | String contains text |
| `hasItem(value)` | Collection contains one item |
| `hasItems(values...)` | Collection contains several items |
| `hasSize(valueOrMatcher)` | Collection has the expected size |
| `everyItem(matcher)` | Every collection item matches |
| `greaterThan(value)` / `lessThan(value)` | Numeric comparison |
| `notNullValue()` / `nullValue()` | Null check |

> [!caution] Numeric Types Matter
> A JSON decimal may be read as `float`, so `equalTo(12.12)` can fail while `equalTo(12.12f)` passes. Configure number parsing or use the expected Java numeric type.

### JSON Path

REST Assured body paths use Groovy GPath syntax. This is not the same implementation as Jayway JsonPath.

For this response:

```json
{
  "users": [
    {"id": 1, "name": "Ana", "active": true},
    {"id": 2, "name": "Sam", "active": false}
  ]
}
```

Useful paths are:

| Path | Result |
|---|---|
| `users[0].name` | `Ana` |
| `users.id` | `[1, 2]` |
| `users.find { it.active }.name` | `Ana` |
| `users.findAll { it.active }.size()` | `1` |

Example validation:

```java
when()
    .get("/users")
.then()
    .body("users.id", hasItems(1, 2))
    .body("users.findAll { it.active }.size()", equalTo(1));
```

### Extracting Response Data

Use extraction when a later step needs data from the response.

```java
Response response = given()
    .baseUri("https://api.example.com")
    .contentType(ContentType.JSON)
    .body("{\"name\":\"Sam\"}")
.when()
    .post("/users")
.then()
    .statusCode(201)
    .extract()
    .response();

int userId = response.path("id");
String location = response.header("Location");
```

Common extraction methods:

| Method | Returns |
|---|---|
| `extract().response()` | Full `Response` object |
| `extract().path("id")` | Value at a body path |
| `response.asString()` | Body as `String` |
| `response.jsonPath()` | `JsonPath` helper |
| `response.as(Type.class)` | Deserialized Java object |
| `response.statusCode()` | Status code |
| `response.header(name)` | Header value |

### Response Time

```java
when()
    .get("/health")
.then()
    .time(lessThan(2_000L));
```

This is useful as a simple guard, but it is not a performance test. The result includes network time, REST Assured processing, JVM warm-up, and environment load.

---

## Authentication

REST Assured supports Basic, Digest, form, OAuth, certificate-based, and custom authentication. The two common beginner cases are Basic authentication and bearer tokens.

### Preemptive Basic Authentication

Preemptive Basic authentication sends credentials with the first request:

```java
given()
    .auth().preemptive().basic(username, password)
.when()
    .get("/admin")
.then()
    .statusCode(200);
```

With challenged Basic authentication, the client first receives an authentication challenge and then sends credentials:

```java
given()
    .auth().basic(username, password)
.when()
    .get("/admin")
.then()
    .statusCode(200);
```

### Bearer Token

Use OAuth 2 bearer authentication:

```java
String token = System.getenv("API_TOKEN");

given()
    .auth().oauth2(token)
.when()
    .get("/profile")
.then()
    .statusCode(200);
```

The direct header form is equivalent for a bearer token:

```java
given()
    .header("Authorization", "Bearer " + token)
.when()
    .get("/profile");
```

> [!danger] Keep Secrets Outside the Code
> Never commit real passwords, access tokens, or API keys. Read them from environment variables or a secret manager, and hide them from logs and reports.

---

## Serialization and Deserialization

Serialization converts a Java object to JSON or XML. Deserialization converts a response body to a Java object.

REST Assured needs a compatible object mapper, such as Jackson or Gson, on the classpath for object mapping.

### Serialization

```java
public record CreateUserRequest(String name, String job) {}
```

```java
CreateUserRequest request = new CreateUserRequest("Sam", "QA Engineer");

given()
    .baseUri("https://api.example.com")
    .contentType(ContentType.JSON)
    .body(request)
.when()
    .post("/users")
.then()
    .statusCode(201);
```

**Advantage:** model classes reduce manual JSON strings and make refactoring safer.

### Deserialization

```java
public record UserResponse(int id, String name, String job) {}
```

```java
UserResponse user = given()
    .baseUri("https://api.example.com")
.when()
    .get("/users/42")
.then()
    .statusCode(200)
    .extract()
    .as(UserResponse.class);

assertEquals(42, user.id());
```

Model field names and types must match the response, unless the object mapper is configured with explicit mappings.

---

## Request and Response Specifications

Specifications store repeated setup or repeated expectations. They reduce duplication and keep tests focused on scenarios.

### RequestSpecification

```java
RequestSpecification requestSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setBasePath("/v1")
    .setContentType(ContentType.JSON)
    .addHeader("Accept", "application/json")
    .build();
```

Use it in a test:

```java
given()
    .spec(requestSpec)
    .pathParam("id", 42)
.when()
    .get("/users/{id}");
```

### ResponseSpecification

```java
ResponseSpecification successSpec = new ResponseSpecBuilder()
    .expectStatusCode(200)
    .expectContentType(ContentType.JSON)
    .build();
```

```java
given()
    .spec(requestSpec)
.when()
    .get("/users/42")
.then()
    .spec(successSpec)
    .body("id", equalTo(42));
```

> [!caution] Do Not Hide the Scenario
> Put stable technical defaults in specifications. Keep values that explain the scenario, such as a special role or invalid input, visible in the test.

---

## Logging and Failure Analysis

REST Assured can log requests and responses:

```java
given()
    .log().all()
    .baseUri("https://api.example.com")
.when()
    .get("/users/42")
.then()
    .log().ifValidationFails()
    .statusCode(200);
```

Useful options:

| Method | Purpose |
|---|---|
| `log().all()` | Logs all available request or response data |
| `log().headers()` | Logs headers |
| `log().body()` | Logs the body |
| `log().params()` | Logs request parameters |
| `log().ifValidationFails()` | Logs only after a validation failure |
| `enableLoggingOfRequestAndResponseIfValidationFails()` | Enables failure logging globally |

When a test fails, check in this order:

1. Confirm the HTTP method and final URL.
2. Check path parameters, query parameters, headers, authentication, and body.
3. Read the status code, response headers, and error body.
4. Compare the response with the API contract.
5. Check business side effects in a database, message broker, or another API when the scenario requires them.
6. Use a trace or correlation ID to find related server logs.

> [!caution] Logging Can Leak Secrets
> `log().all()` may print authorization headers, cookies, passwords, or personal data. Use full logging carefully and mask sensitive values in shared reports.

---

## JSON Schema Validation

JSON Schema validation checks the response structure, required fields, and field types.

Add the optional module:

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>${rest-assured.version}</version>
    <scope>test</scope>
</dependency>
```

Place `user-schema.json` in `src/test/resources/schemas/`, then validate it:

```java
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

when()
    .get("/users/42")
.then()
    .statusCode(200)
    .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
```

> [!info] Schema vs Business Assertion
> A schema can prove that `id` is an integer and `name` is present. It cannot prove that the returned user is the correct user or that a balance was calculated correctly. Use schema checks together with business assertions.

---

## Framework Structure

In a maintainable framework, tests should not repeat full low-level request setup. A common structure is:

```text
src/test/java
├── api
│   ├── client
│   │   └── UserApi.java
│   ├── model
│   │   ├── CreateUserRequest.java
│   │   └── UserResponse.java
│   ├── specification
│   │   └── ApiSpecifications.java
│   └── test
│       └── UserApiTest.java
└── support
    └── TestConfig.java
```

- **Client layer** sends requests and returns responses or models.
- **Model layer** stores request and response data.
- **Specification layer** stores stable shared HTTP setup.
- **Test layer** describes scenarios and business assertions.
- **Configuration layer** reads URLs, credentials, and environment settings.

Example client:

```java
public final class UserApi {
    private final RequestSpecification specification;

    public UserApi(RequestSpecification specification) {
        this.specification = specification;
    }

    public Response getUser(int id) {
        return given()
            .spec(specification)
            .pathParam("id", id)
        .when()
            .get("/users/{id}");
    }
}
```

Example test:

```java
@Test
void shouldReturnExistingUser() {
    userApi.getUser(42)
        .then()
        .statusCode(200)
        .body("id", equalTo(42));
}
```

This follows the separation principles from [[20 Automation Framework Architecture]].

---

## Common Mistakes

| Mistake | Why it is a problem | Better approach |
|---|---|---|
| Checking only `statusCode(200)` | A wrong body can still return 200 | Check contract and business fields |
| Hardcoding tokens and passwords | Secrets can leak through Git or reports | Use environment variables or a secret manager |
| Using the wrong parameter type | The server receives a different URL or body | Separate path, query, and form parameters |
| Forgetting `Content-Type` | The server may reject or misread the body | Set the expected media type |
| Comparing the entire dynamic body | IDs, dates, and order can make the test unstable | Assert important stable fields |
| Repeating base URI and headers | Setup becomes difficult to maintain | Use a `RequestSpecification` |
| Hiding all data in specifications | The scenario becomes hard to understand | Keep scenario-specific values in the test |
| Sharing mutable global configuration | Parallel tests can affect one another | Prefer stable specifications or isolated setup |
| Logging everything in CI | Logs may contain secrets and personal data | Log on failure and mask sensitive fields |
| Treating a time assertion as a load test | One request does not measure system capacity | Use a performance testing tool for load |
| Ignoring side effects | The HTTP response may look correct while data is wrong | Check the database, event, or next API state when needed |

---

## Interview Questions

### Beginner Questions

**1. What is REST Assured?**

REST Assured is a Java DSL for sending HTTP requests and validating API responses in automated tests.

**2. What do `given()`, `when()`, and `then()` mean?**

`given()` prepares the request, `when()` performs it, and `then()` validates the response.

**3. What is the difference between a path parameter and a query parameter?**

A path parameter is part of the URL path that points to a specific resource (`/users/42`). A query parameter comes after `?` and filters or modifies the representation (`/users?role=admin`).

**4. What library provides matchers such as `equalTo()` and `hasItem()`?**

Hamcrest provides these matchers. REST Assured uses them for readable response assertions.

**5. Is REST Assured a test runner?**

No. JUnit 5 or TestNG runs the tests. REST Assured handles HTTP requests and response validation.

### Intermediate Questions

**1. Why use a `RequestSpecification`?**

It stores repeated technical setup such as base URI, content type, and common headers. This removes duplication.

**2. What is the difference between validation and extraction?**

Validation checks that a response matches expectations. Extraction reads data from the response for later steps.

**3. What is serialization in an API test?**

Serialization converts a Java object into a request format such as JSON. Deserialization converts a response into a Java object.

**4. What is the difference between preemptive and challenged Basic authentication?**

Preemptive Basic authentication sends credentials immediately. Challenged Basic authentication waits for the server's authentication challenge first.

**5. Why should a test check more than the status code?**

A correct status code does not prove that headers, body fields, business rules, or side effects are correct.

### Advanced Questions

**1. Why can mutable global REST Assured configuration be risky?**

Parallel tests can change the same global state and affect one another. Stable specifications or isolated setup are safer.

**2. Why is JSON Schema validation not enough by itself?**

It checks response structure and types, but it does not prove that business values are correct.

**3. Why is a REST Assured response-time assertion not a performance test?**

It measures one request and includes network, client, JVM, and environment overhead. It does not measure capacity under controlled load.

**4. How should an API test verify side effects?**

After validating the response, it should check the relevant database record, message, or later API state when the scenario requires it.

**5. Where should REST Assured code live in an automation framework?**

Low-level requests usually belong in an API client layer. Models hold data, specifications hold stable setup, and tests hold scenarios and business assertions.

### Code Questions

**1. What does this test send and validate?**

```java
given()
    .baseUri("https://api.example.com")
    .pathParam("id", 42)
.when()
    .get("/users/{id}")
.then()
    .statusCode(200)
    .body("id", equalTo(42));
```

**Answer:** It sends `GET https://api.example.com/users/42`, expects status 200, and checks that the response field `id` equals 42.

**2. What is the final URL?**

```java
given()
    .baseUri("https://api.example.com")
    .pathParam("id", 42)
    .queryParam("include", "roles")
.when()
    .get("/users/{id}");
```

**Answer:** The final URL is `https://api.example.com/users/42?include=roles`.

**3. What is stored in `userId`?**

```java
int userId = given()
    .contentType(ContentType.JSON)
    .body("{\"name\":\"Sam\"}")
.when()
    .post("/users")
.then()
    .statusCode(201)
    .extract()
    .path("id");
```

**Answer:** `userId` stores the value of the `id` field from the response body. The exact number depends on the server response.

**4. What security problem does this code have?**

```java
given()
    .log().all()
    .header("Authorization", "Bearer real-production-token")
.when()
    .get("/profile");
```

**Answer:** The token is hardcoded and full request logging may print it. Read the token from secure configuration and mask it in logs.
