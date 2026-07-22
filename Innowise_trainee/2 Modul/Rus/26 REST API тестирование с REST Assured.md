# REST API тестирование с REST Assured

## Содержание

- [[#Зачем нужен REST Assured]]
- [[#Настройка проекта]]
	- [[#Maven]]
	- [[#Gradle]]
	- [[#Статические импорты]]
- [[#Структура Given When Then]]
	- [[#Given]]
	- [[#When]]
	- [[#Then]]
- [[#Отправка запросов]]
	- [[#HTTP-методы]]
	- [[#Методы настройки запроса]]
	- [[#Query parameter и Path parameter]]
	- [[#Headers Cookies и Content Type]]
	- [[#Request Body]]
- [[#Проверка ответов]]
	- [[#Status Headers и Content Type]]
	- [[#Response Body и Hamcrest Matchers]]
	- [[#JSON Path]]
	- [[#Извлечение данных из Response]]
	- [[#Response Time]]
- [[#Authentication]]
	- [[#Preemptive Basic Authentication]]
	- [[#Bearer Token]]
- [[#Сериализация и десериализация]]
	- [[#Сериализация]]
	- [[#Десериализация]]
- [[#Request и Response Specifications]]
	- [[#RequestSpecification]]
	- [[#ResponseSpecification]]
- [[#Logging и анализ падений]]
- [[#JSON Schema Validation]]
- [[#Структура фреймворка]]
- [[#Типичные ошибки]]
- [[#Вопросы для собеседования]]
	- [[#Вопросы начального уровня]]
	- [[#Вопросы среднего уровня]]
	- [[#Вопросы повышенного уровня]]
	- [[#Вопросы с кодом]]

**Связанные заметки:** [[AQA Java rus]]

> [!caution] Нет в видео-курсе
> Курсы Заура заканчиваются на Java Core — REST Assured не разбирается. Это AQA-надстройка: учись по этой заметке и практике, а при изменениях API используй официальную документацию.

---

## Зачем нужен REST Assured

**REST Assured** — это Java DSL для тестирования HTTP API. Он отправляет requests, читает responses и проверяет результат в automated tests.

**Цель:** сделать API tests понятными, сохранив полный контроль над HTTP request и response.

REST Assured поддерживает основные HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD` и `OPTIONS`.

Он позволяет проверять:

- status codes
- headers и cookies
- JSON и XML bodies
- response time
- работу authentication
- JSON Schema
- business rules и side effects

> [!info] Library и Test Framework
> REST Assured — это HTTP testing library. JUnit 5 или TestNG находит и запускает tests, а REST Assured отправляет requests и проверяет responses.

---

## Настройка проекта

В этом модуле используется Java 21, поэтому можно применять REST Assured 6.x. Для REST Assured 6 нужна Java 17 или новее.

### Maven

Добавь REST Assured со scope `test`:

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

Размещай dependency REST Assured перед JUnit, если обе библиотеки добавляют свои версии Hamcrest в проект.

### Gradle

```groovy
dependencies {
    testImplementation 'io.rest-assured:rest-assured:6.0.1'
}
```

### Статические импорты

Static imports делают DSL короче:

```java
import static io.restassured.RestAssured.*;
import static io.restassured.matcher.RestAssuredMatchers.*;
import static org.hamcrest.Matchers.*;
```

Основная dependency `rest-assured` уже включает модули REST Assured `JsonPath` и `XmlPath`.

---

## Структура Given When Then

Рекомендуемый стиль REST Assured использует BDD-подобную структуру `given–when–then`.

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

**Определение:** `given()` создаёт request specification.

Используй его, чтобы подготовить:

- base URI и base path
- parameters
- headers и cookies
- authentication
- content type
- request body
- logging и filters

### When

**Определение:** `when()` отделяет подготовку request от его выполнения.

HTTP method после `when()` отправляет request:

```java
when().get("/users");
```

### Then

**Определение:** `then()` создаёт validatable response.

Используй его для проверки status, headers, body и response time:

```java
then()
    .statusCode(200)
    .contentType(ContentType.JSON);
```

> [!tip] Arrange Act Assert и Given When Then
> `given` подготавливает test data, `when` выполняет request, а `then` проверяет result. Эта структура похожа на Arrange–Act–Assert из unit testing.

---

## Отправка запросов

### HTTP-методы

| Метод | Обычное назначение | Вызов REST Assured |
|---|---|---|
| `GET` | Получить resource | `get("/users/42")` |
| `POST` | Создать resource или запустить operation | `post("/users")` |
| `PUT` | Полностью заменить resource | `put("/users/42")` |
| `PATCH` | Частично изменить resource | `patch("/users/42")` |
| `DELETE` | Удалить resource | `delete("/users/42")` |
| `HEAD` | Получить headers без response body | `head("/users/42")` |
| `OPTIONS` | Получить поддерживаемые варианты взаимодействия | `options("/users")` |
| Нестандартный метод | Отправить другой HTTP method | `request("CONNECT", "/proxy")` |

### Методы настройки запроса

| Метод | Назначение | Пример |
|---|---|---|
| `baseUri()` | Задаёт scheme, host и необязательный port | `baseUri("https://api.example.com")` |
| `basePath()` | Задаёт общий префикс path | `basePath("/v1")` |
| `queryParam()` | Добавляет query parameter | `queryParam("page", 2)` |
| `pathParam()` | Подставляет значение в path placeholder | `pathParam("id", 42)` |
| `formParam()` | Добавляет HTML form parameter | `formParam("name", "Sam")` |
| `header()` / `headers()` | Добавляет один или несколько headers | `header("X-Trace-Id", traceId)` |
| `cookie()` / `cookies()` | Добавляет один или несколько cookies | `cookie("session", sessionId)` |
| `contentType()` | Задаёт `Content-Type` request | `contentType(ContentType.JSON)` |
| `accept()` | Задаёт ожидаемый media type response | `accept(ContentType.JSON)` |
| `body()` | Добавляет request body | `body(json)` |
| `multiPart()` | Добавляет multipart form data или file | `multiPart("file", file)` |
| `spec()` | Повторно использует request specification | `spec(requestSpec)` |

### Query parameter и Path parameter

**Path parameter** — часть URL path, которая указывает на конкретный resource. Без него ресурс не существует.

```text
GET /users/42          → пользователь с id = 42
GET /orders/15/items/3 → item 3 в заказе 15
```

**Query parameter** — идёт после `?`, уточняет или фильтрует возвращаемое представление. Ресурс тот же, меняется только то, что вернётся.

```text
GET /users?role=admin&active=true → все юзеры, но только admin и active
GET /products?sort=price&page=2   → товары, страница 2, сортировка по цене
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

Итоговый URL:

```text
https://api.example.com/users/42?include=roles
```

> [!tip] Path parameter и Query parameter
> Path = «какой ресурс взять». Query = «как его показать» (фильтр, сортировка, пагинация).

### Headers Cookies и Content Type

```java
given()
    .header("X-Trace-Id", "test-123")
    .cookie("locale", "en")
    .contentType(ContentType.JSON)
    .accept(ContentType.JSON)
.when()
    .get("/profile");
```

- `Content-Type` описывает формат request body.
- `Accept` описывает форматы response, которые может принять client.
- Cookies часто хранят session или настройки.
- Custom headers могут передавать trace ID, API version или feature flags.

### Request Body

Request body может быть `String`, `Map`, Java object, массивом bytes или file.

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

> [!caution] Указывай Content Type
> Строки, похожей на JSON, недостаточно. Укажи `Content-Type: application/json`, если server ожидает JSON.

---

## Проверка ответов

Полезный API test проверяет больше одной transport detail. Он должен проверять contract и важный для scenario business result.

### Status Headers и Content Type

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

| Метод | Что проверяет |
|---|---|
| `statusCode()` | Числовой HTTP status code |
| `statusLine()` | Полную status line |
| `header()` / `headers()` | Response headers |
| `cookie()` / `cookies()` | Response cookies |
| `contentType()` | Media type response |

### Response Body и Hamcrest Matchers

REST Assured использует Hamcrest matchers для понятных assertions.

```java
when()
    .get("/users/42")
.then()
    .body("id", equalTo(42))
    .body("name", not(isEmptyOrNullString()))
    .body("roles", hasItem("admin"))
    .body("roles", hasSize(greaterThan(0)));
```

| Matcher | Назначение |
|---|---|
| `equalTo(value)` / `is(value)` | Точное равенство |
| `not(matcher)` | Отрицание другого matcher |
| `containsString(text)` | String содержит текст |
| `hasItem(value)` | Collection содержит один item |
| `hasItems(values...)` | Collection содержит несколько items |
| `hasSize(valueOrMatcher)` | Collection имеет ожидаемый размер |
| `everyItem(matcher)` | Каждый item в collection соответствует matcher |
| `greaterThan(value)` / `lessThan(value)` | Сравнение чисел |
| `notNullValue()` / `nullValue()` | Проверка на `null` |

> [!caution] Типы чисел важны
> Десятичное число из JSON может быть прочитано как `float`, поэтому `equalTo(12.12)` может упасть, а `equalTo(12.12f)` — пройти. Настрой parsing чисел или используй ожидаемый Java numeric type.

### JSON Path

В body paths REST Assured используется Groovy GPath syntax. Это не та же реализация, что Jayway JsonPath.

Для такого response:

```json
{
  "users": [
    {"id": 1, "name": "Ana", "active": true},
    {"id": 2, "name": "Sam", "active": false}
  ]
}
```

Полезные paths:

| Path | Результат |
|---|---|
| `users[0].name` | `Ana` |
| `users.id` | `[1, 2]` |
| `users.find { it.active }.name` | `Ana` |
| `users.findAll { it.active }.size()` | `1` |

Пример проверки:

```java
when()
    .get("/users")
.then()
    .body("users.id", hasItems(1, 2))
    .body("users.findAll { it.active }.size()", equalTo(1));
```

### Извлечение данных из Response

Используй extraction, если данные из response нужны на следующем шаге.

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

Основные extraction methods:

| Метод | Возвращает |
|---|---|
| `extract().response()` | Полный object `Response` |
| `extract().path("id")` | Значение по body path |
| `response.asString()` | Body как `String` |
| `response.jsonPath()` | Helper `JsonPath` |
| `response.as(Type.class)` | Десериализованный Java object |
| `response.statusCode()` | Status code |
| `response.header(name)` | Значение header |

### Response Time

```java
when()
    .get("/health")
.then()
    .time(lessThan(2_000L));
```

Такая проверка полезна как простой guard, но это не performance test. Результат включает network time, обработку REST Assured, JVM warm-up и нагрузку environment.

---

## Authentication

REST Assured поддерживает Basic, Digest, form, OAuth, certificate-based и custom authentication. Два частых начальных случая — Basic authentication и bearer tokens.

### Preemptive Basic Authentication

Preemptive Basic authentication отправляет credentials с первым request:

```java
given()
    .auth().preemptive().basic(username, password)
.when()
    .get("/admin")
.then()
    .statusCode(200);
```

При challenged Basic authentication client сначала получает authentication challenge, а затем отправляет credentials:

```java
given()
    .auth().basic(username, password)
.when()
    .get("/admin")
.then()
    .statusCode(200);
```

### Bearer Token

Использование OAuth 2 bearer authentication:

```java
String token = System.getenv("API_TOKEN");

given()
    .auth().oauth2(token)
.when()
    .get("/profile")
.then()
    .statusCode(200);
```

Для bearer token можно использовать равнозначную прямую передачу header:

```java
given()
    .header("Authorization", "Bearer " + token)
.when()
    .get("/profile");
```

> [!danger] Храни секреты вне кода
> Никогда не добавляй настоящие passwords, access tokens или API keys в Git. Читай их из environment variables или secret manager и скрывай в logs и reports.

---

## Сериализация и десериализация

Serialization преобразует Java object в JSON или XML. Deserialization преобразует response body в Java object.

Для object mapping REST Assured нужен совместимый object mapper, например Jackson или Gson, в classpath.

### Сериализация

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

**Преимущество:** model classes уменьшают количество ручных JSON strings и делают refactoring безопаснее.

### Десериализация

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

Имена и типы model fields должны соответствовать response, если в object mapper не настроен явный mapping.

---

## Request и Response Specifications

Specifications хранят повторяющиеся настройки или ожидания. Они уменьшают дублирование и помогают tests фокусироваться на scenarios.

### RequestSpecification

```java
RequestSpecification requestSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setBasePath("/v1")
    .setContentType(ContentType.JSON)
    .addHeader("Accept", "application/json")
    .build();
```

Использование в test:

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

> [!caution] Не скрывай scenario
> Помещай стабильные технические defaults в specifications. Значения, которые объясняют scenario, например особую role или invalid input, оставляй видимыми в test.

---

## Logging и анализ падений

REST Assured умеет логировать requests и responses:

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

Полезные options:

| Метод | Назначение |
|---|---|
| `log().all()` | Логирует все доступные данные request или response |
| `log().headers()` | Логирует headers |
| `log().body()` | Логирует body |
| `log().params()` | Логирует request parameters |
| `log().ifValidationFails()` | Логирует только после validation failure |
| `enableLoggingOfRequestAndResponseIfValidationFails()` | Глобально включает logging при падении validation |

Когда test падает, проверяй в таком порядке:

1. Подтверди HTTP method и итоговый URL.
2. Проверь path parameters, query parameters, headers, authentication и body.
3. Прочитай status code, response headers и error body.
4. Сравни response с API contract.
5. Проверь business side effects в database, message broker или другом API, если этого требует scenario.
6. Используй trace ID или correlation ID для поиска связанных server logs.

> [!caution] Logging может раскрыть секреты
> `log().all()` может вывести authorization headers, cookies, passwords или personal data. Осторожно используй полный logging и маскируй sensitive values в общих reports.

---

## JSON Schema Validation

JSON Schema validation проверяет структуру response, required fields и field types.

Добавь optional module:

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>${rest-assured.version}</version>
    <scope>test</scope>
</dependency>
```

Помести `user-schema.json` в `src/test/resources/schemas/`, затем выполни validation:

```java
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

when()
    .get("/users/42")
.then()
    .statusCode(200)
    .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
```

> [!info] Schema и Business Assertion
> Schema может подтвердить, что `id` является integer, а `name` присутствует. Она не доказывает, что возвращён правильный user или что balance рассчитан правильно. Используй schema checks вместе с business assertions.

---

## Структура фреймворка

В поддерживаемом framework tests не должны повторять полный low-level request setup. Часто используется такая структура:

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

- **Client layer** отправляет requests и возвращает responses или models.
- **Model layer** хранит request и response data.
- **Specification layer** хранит стабильный общий HTTP setup.
- **Test layer** описывает scenarios и business assertions.
- **Configuration layer** читает URLs, credentials и environment settings.

Пример client:

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

Пример test:

```java
@Test
void shouldReturnExistingUser() {
    userApi.getUser(42)
        .then()
        .statusCode(200)
        .body("id", equalTo(42));
}
```

Эта структура следует принципам разделения из [[20 Архитектура фреймворка автотестов]].

---

## Типичные ошибки

| Ошибка | Почему это проблема | Что делать лучше |
|---|---|---|
| Проверять только `statusCode(200)` | Неправильный body тоже может вернуться с 200 | Проверяй contract и business fields |
| Хранить tokens и passwords в коде | Секреты могут попасть в Git или reports | Используй environment variables или secret manager |
| Использовать неправильный parameter type | Server получает другой URL или body | Разделяй path, query и form parameters |
| Забывать `Content-Type` | Server может отклонить или неправильно прочитать body | Указывай ожидаемый media type |
| Сравнивать весь dynamic body | IDs, dates и порядок могут сделать test нестабильным | Проверяй важные стабильные fields |
| Повторять base URI и headers | Setup становится трудно поддерживать | Используй `RequestSpecification` |
| Скрывать все данные в specifications | Scenario становится трудно понять | Оставляй scenario-specific values в test |
| Использовать общий mutable global configuration | Parallel tests могут влиять друг на друга | Используй стабильные specifications или изолированный setup |
| Логировать всё в CI | Logs могут содержать секреты и personal data | Логируй при падении и маскируй sensitive fields |
| Считать time assertion нагрузочным тестом | Один request не измеряет capacity системы | Используй performance testing tool для load |
| Игнорировать side effects | HTTP response может быть правильным, а данные — нет | При необходимости проверяй database, event или следующее API state |

---

## Вопросы для собеседования

### Вопросы начального уровня

**1. Что такое REST Assured?**

REST Assured — это Java DSL для отправки HTTP requests и проверки API responses в automated tests.

**2. Что означают `given()`, `when()` и `then()`?**

`given()` подготавливает request, `when()` выполняет его, а `then()` проверяет response.

**3. В чём разница между path parameter и query parameter?**

Path parameter — часть URL path, указывает на конкретный ресурс (`/users/42`). Query parameter — идёт после `?`, фильтрует или изменяет представление (`/users?role=admin`).

**4. Какая библиотека предоставляет matchers `equalTo()` и `hasItem()`?**

Эти matchers предоставляет Hamcrest. REST Assured использует их для понятных response assertions.

**5. Является ли REST Assured test runner?**

Нет. Tests запускает JUnit 5 или TestNG. REST Assured отвечает за HTTP requests и response validation.

### Вопросы среднего уровня

**1. Зачем использовать `RequestSpecification`?**

Он хранит повторяющийся технический setup: base URI, content type и общие headers. Это уменьшает дублирование.

**2. В чём разница между validation и extraction?**

Validation проверяет, что response соответствует ожиданиям. Extraction читает данные из response для следующих шагов.

**3. Что такое serialization в API test?**

Serialization преобразует Java object в формат request, например JSON. Deserialization преобразует response в Java object.

**4. В чём разница между preemptive и challenged Basic authentication?**

Preemptive Basic authentication отправляет credentials сразу. Challenged Basic authentication сначала ждёт authentication challenge от server.

**5. Почему test должен проверять не только status code?**

Правильный status code не доказывает, что headers, body fields, business rules и side effects правильные.

### Вопросы повышенного уровня

**1. Чем опасен mutable global configuration REST Assured?**

Parallel tests могут изменять общее состояние и влиять друг на друга. Стабильные specifications или изолированный setup безопаснее.

**2. Почему одной JSON Schema validation недостаточно?**

Она проверяет структуру и типы response, но не доказывает правильность business values.

**3. Почему проверка response time в REST Assured не является performance test?**

Она измеряет один request и включает network, client, JVM и environment overhead. Она не измеряет capacity под контролируемой load.

**4. Как API test должен проверять side effects?**

После response validation он должен проверить нужную запись в database, message или следующее API state, если этого требует scenario.

**5. Где должен находиться код REST Assured в automation framework?**

Low-level requests обычно находятся в API client layer. Models хранят данные, specifications — стабильный setup, а tests — scenarios и business assertions.

### Вопросы с кодом

**1. Какой request отправляет этот test и что он проверяет?**

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

**Ответ:** Он отправляет `GET https://api.example.com/users/42`, ожидает status 200 и проверяет, что field `id` в response равен 42.

**2. Каким будет итоговый URL?**

```java
given()
    .baseUri("https://api.example.com")
    .pathParam("id", 42)
    .queryParam("include", "roles")
.when()
    .get("/users/{id}");
```

**Ответ:** Итоговый URL — `https://api.example.com/users/42?include=roles`.

**3. Что будет сохранено в `userId`?**

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

**Ответ:** В `userId` будет сохранено значение field `id` из response body. Точное число зависит от response server.

**4. Какая security problem есть в этом коде?**

```java
given()
    .log().all()
    .header("Authorization", "Bearer real-production-token")
.when()
    .get("/profile");
```

**Ответ:** Token записан прямо в коде, а полный request logging может его вывести. Token нужно читать из secure configuration и маскировать в logs.
