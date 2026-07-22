# Паттерны проектирования для AQA

## Содержание

- [[#Зачем нужны паттерны проектирования]]
- [[#Категории паттернов]]
	- [[#Creational patterns]]
	- [[#Structural patterns]]
	- [[#Behavioral patterns]]
- [[#Factory Pattern]]
	- [[#Simple Factory]]
	- [[#Factory Method]]
- [[#Builder Pattern]]
	- [[#Test Data Builder]]
	- [[#Request Specification Builder]]
- [[#Strategy Pattern]]
	- [[#Authentication Strategy]]
- [[#Выбор паттерна]]
- [[#Совместное использование паттернов в AQA-фреймворке]]
- [[#Типичные ошибки]]
- [[#Вопросы для собеседования]]
	- [[#Вопросы начального уровня]]
	- [[#Вопросы среднего уровня]]
	- [[#Вопросы повышенного уровня]]
	- [[#Вопросы с кодом]]

**Связанные заметки:** [[AQA Java rus]]

> [!caution] Нет в видео-курсе
> Курсы Заура заканчиваются на Java Core — паттерны проектирования для AQA не разбираются. Это AQA-надстройка: учись по этой заметке и практике и применяй паттерн только тогда, когда он решает реальную проблему проектирования.

---

## Зачем нужны паттерны проектирования

**Паттерн проектирования** — это повторно используемая идея решения распространённой software problem. Он описывает взаимодействие classes и objects, но не является готовым кодом, который нужно копировать без изменений.

**Цель:** упростить расширение, тестирование и поддержку кода с помощью решений, понятных другим разработчикам.

В AQA framework паттерны помогают:

- создавать browser drivers или API clients
- собирать test data с понятными default values
- изменять behavior во время выполнения
- отделять test scenarios от технического setup
- уменьшать большие блоки `if` и `switch`

Паттерны полезны, когда изменение или дублирование уже существует. Для небольшой проблемы часто достаточно прямого constructor, одного метода или простого condition.

> [!danger] Сначала решай проблему
> Не добавляй паттерн только потому, что он популярен. Паттерн должен устранять реальную проблему, а не добавлять новые classes без пользы.

---

## Категории паттернов

Классические категории показывают, какой вид design problem решает паттерн.

### Creational patterns

**Определение:** Creational patterns управляют созданием objects.

**Цель:** скрыть сложное создание и уменьшить прямую зависимость от concrete classes.

Основные примеры:

- Factory Method
- Abstract Factory
- Builder
- Prototype
- Singleton

В AQA часто применяются Factory и Builder для drivers, clients, configuration и test data.

### Structural patterns

**Определение:** Structural patterns объединяют classes и objects в более крупные структуры.

**Цель:** связывать components без сильной зависимости между ними.

Основные примеры:

- Adapter
- Decorator
- Facade
- Proxy
- Composite

В AQA Adapter может скрывать различия между сторонними tools, а Facade — предоставлять простой interface для сложной subsystem.

### Behavioral patterns

**Определение:** Behavioral patterns организуют communication и responsibilities между objects.

**Цель:** сделать behavior заменяемым и убрать decision logic из больших conditions.

Основные примеры:

- Strategy
- Observer
- Command
- Template Method
- Chain of Responsibility

В AQA Strategy может выбирать authentication, retry, ожидание или data generation behavior во время выполнения.

> [!tip] Краткое сравнение
> | Категория | Главный вопрос | Паттерн в этой заметке | Пример в AQA |
> |---|---|---|---|
> | Creational | Как создаётся object? | Factory, Builder | Создание driver или test user |
> | Structural | Как objects связаны? | Подробно не рассматривается | Адаптация reporting library |
> | Behavioral | Как можно изменить behavior? | Strategy | Выбор способа authentication |

---

## Factory Pattern

**Определение:** Factory централизует создание objects и возвращает object через общий type.

**Цель:** убрать прямое создание concrete implementations из tests и другого client code.

**Как работает:** caller передаёт type или option. Factory выбирает concrete class, создаёт его и возвращает interface или base type.

**Преимущества:**

- скрывает детали создания
- убирает зависимость tests от concrete implementations
- хранит creation logic в одном месте
- упрощает добавление новых implementations

**Недостатки:**

- добавляет дополнительную abstraction и новые classes
- может превратиться в большой conditional, если отвечает за слишком много задач
- не отменяет необходимость настраивать и закрывать созданные resources

### Simple Factory

Simple Factory — это class или method, который выбирает implementation, часто с помощью `switch`. Его часто называют Factory, но он не является GoF Factory Method pattern.

Пример создания browser:

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

Test зависит от `WebDriver`, а не напрямую от одного concrete driver class. Factory выбирает implementation.

### Factory Method

Factory Method определяет creation method в base type и позволяет subclasses выбирать concrete product.

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

**Когда использовать:** применяй Factory Method, когда создание должно изменяться через inheritance или каждому provider нужны собственные creation steps. Для простого выбора browser обычно достаточно Simple Factory.

> [!tip] Simple Factory и Factory Method
> | Пункт | Simple Factory | Factory Method |
> |---|---|---|
> | Основной механизм | Один method выбирает implementation | Subclasses переопределяют creation method |
> | GoF pattern | Нет | Да |
> | Количество classes | Обычно меньше | Обычно больше |
> | Подходящий пример в AQA | Простой выбор driver или client | Providers с разным setup logic |

---

## Builder Pattern

**Определение:** Builder создаёт сложный object пошагово и возвращает итоговый object через `build()`.

**Цель:** сделать создание object понятным, когда есть много optional values, default values или validation rules.

**Как работает:** Builder хранит временные values. Fluent methods изменяют выбранные values и возвращают тот же builder. `build()` проверяет state и создаёт итоговый object.

**Преимущества:**

- делает test data понятными
- поддерживает полезные default values
- заменяет constructors с большим количеством parameters
- позволяет повторно использовать разные object variations
- может проверять data до создания object

**Недостатки:**

- добавляет лишний код для простых objects
- плохие defaults могут скрывать важные test data
- mutable builder нельзя использовать совместно между parallel tests

### Test Data Builder

Предположим, test нужны данные user:

```java
public record UserData(
        String name,
        String email,
        String role,
        boolean active
) {
}
```

Builder задаёт valid defaults и позволяет каждому test изменять только важные fields:

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

Test ясно показывает, чем отличается этот user. Остальные valid fields берутся из defaults.

### Request Specification Builder

В REST Assured есть реальный пример Builder:

```java
RequestSpecification specification = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setBasePath("/v1")
    .setContentType(ContentType.JSON)
    .addHeader("Accept", "application/json")
    .build();
```

Каждый method настраивает одну часть. `build()` возвращает итоговый `RequestSpecification`.

> [!tip] Builder и Constructor
> | Пункт | Constructor | Builder |
> |---|---|---|
> | Лучше подходит для | Нескольких required values | Многих optional values или defaults |
> | Читаемость | Значение parameters может быть непонятно | Имена fluent methods объясняют values |
> | Validation | Обычно внутри constructor | Часто внутри `build()` |
> | Дополнительный код | Мало | Больше |

---

## Strategy Pattern

**Определение:** Strategy помещает заменяемый behavior за общий interface.

**Цель:** выбирать или изменять algorithm во время выполнения без изменения class, который его использует.

**Как работает:** context получает strategy object и делегирует ему одну responsibility. Разные strategies реализуют один interface по-разному.

**Преимущества:**

- заменяет большие conditions полиморфизмом
- поддерживает выбор behavior во время выполнения
- хранит каждый algorithm в отдельном class
- упрощает отдельное тестирование behavior

**Недостатки:**

- создаёт больше маленьких classes
- caller должен знать, какую strategy выбрать
- не нужен, если существует только один стабильный behavior

### Authentication Strategy

API client может поддерживать разные способы authentication.

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

API client делегирует authentication выбранной strategy:

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

В `ApiClient` не нужен `if (authType == ...)`. Новый authentication method можно добавить как новую strategy.

> [!tip] Strategy и Conditional
> | Пункт | Conditional | Strategy |
> |---|---|---|
> | Лучше подходит для | Одного небольшого стабильного выбора | Нескольких behaviors, которые изменяются или растут |
> | Расширение | Изменить существующий conditional | Добавить новую implementation |
> | Тестирование | Branches проверяются вместе | Каждая strategy проверяется отдельно |
> | Стоимость | Очень низкая | Больше classes и interfaces |

---

## Выбор паттерна

Выбирай паттерн на основе проблемы, а не имени class, который хочется создать.

| Проблема | Подходящий паттерн | Пример в AQA | Более простое решение |
|---|---|---|---|
| Client code не должен знать concrete class | Factory | Создание `ChromeDriver` или `FirefoxDriver` | Прямой constructor для одной implementation |
| Object имеет много optional fields и полезных defaults | Builder | Создание test users или request specifications | Constructor для небольшого value object |
| Один behavior должен заменяться во время выполнения | Strategy | Выбор authentication или retry logic | Небольшой `if` для двух стабильных случаев |

Сначала задай вопросы:

1. Какое изменение сейчас выполнить сложно?
2. Какое дублирование уберёт паттерн?
3. Может ли ту же проблему решить небольшой method или class?
4. Поймёт ли другой разработчик abstraction без длинного объяснения?
5. Можно ли тестировать каждый component отдельно?

---

## Совместное использование паттернов в AQA-фреймворке

Паттерны решают разные проблемы и могут работать вместе:

1. **Factory** выбирает и создаёт нужный `WebDriver`.
2. **Builder** создаёт test data для scenario.
3. **Strategy** выбирает authentication или другой заменяемый behavior.
4. Test использует эти objects, но не содержит детали их создания.

Пример структуры framework:

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

Такое разделение поддерживает принципы framework из [[20 Архитектура фреймворка автотестов]] и design principles из [[23 Инженерные практики для AQA]].

> [!info] У паттернов разные обязанности
> Factory отвечает на вопрос «какой object создать?», Builder — «как собрать сложный object?», а Strategy — «какой behavior использовать?».

---

## Типичные ошибки

| Ошибка | Почему это проблема | Что делать лучше |
|---|---|---|
| Добавлять паттерн до появления реальной потребности | Код становится сложнее, но проблема не исчезает | Начни с простого решения и выполни refactoring при появлении изменений или дублирования |
| Называть каждый creation helper Factory Method | Simple Factory путается с GoF pattern | Называй точный используемый паттерн |
| Помещать setup, logging, waits и cleanup в одну Factory | Factory получает слишком много responsibilities | Оставь Factory только creation logic |
| Использовать Builder для object с двумя required fields | Дополнительный код почти не приносит пользы | Используй constructor или factory method |
| Скрывать важные scenario values в Builder defaults | Test становится трудно понять | Показывай values, которые определяют scenario |
| Использовать один mutable Builder между tests | Parallel tests могут влиять друг на друга | Создавай новый builder для каждого object или test |
| Создавать Strategy classes для одного неизменяемого algorithm | Большое количество classes не добавляет гибкости | Используй одну прямую implementation |
| Оставлять большой `switch` внутри Strategy context | Behavior всё ещё не разделён | Перенеси каждый algorithm в отдельную strategy class |
| Хранить один global static `WebDriver` | Tests делят browser state и ломаются при parallel run | Используй driver с test scope или thread scope |
| Объединять паттерны без чётких границ | Responsibilities скрываются между множеством layers | Дай каждому паттерну одну конкретную проблему |

---

## Вопросы для собеседования

### Вопросы начального уровня

**1. Что такое паттерн проектирования?**

Паттерн проектирования — это повторно используемая идея решения распространённой software problem. Это не готовый код, который всегда нужно копировать без изменений.

**2. Какие три основные категории паттернов существуют?**

Creational patterns управляют созданием objects, Structural patterns связывают objects, а Behavioral patterns организуют behavior и communication.

**3. Какую проблему решает Factory?**

Factory централизует создание objects и скрывает concrete implementations от client code.

**4. Какую проблему решает Builder?**

Builder пошагово создаёт сложные objects, особенно если у них есть optional values или полезные defaults.

**5. Какую проблему решает Strategy?**

Strategy делает behavior заменяемым через общий interface и позволяет выбрать его во время выполнения.

### Вопросы среднего уровня

**1. Почему `BrowserFactory` должен возвращать `WebDriver`, а не `ChromeDriver`?**

Возврат interface убирает зависимость client code от одной browser implementation.

**2. В чём разница между Simple Factory и Factory Method?**

Simple Factory обычно выбирает implementation в одном method. Factory Method позволяет subclasses переопределять creation method.

**3. Когда Builder лучше constructor?**

Builder лучше, когда у object много optional values, defaults или validation rules. Constructor проще для нескольких required values.

**4. Как Strategy поддерживает Open/Closed Principle?**

Новый behavior можно добавить как новую strategy implementation без изменения context class.

**5. Можно ли использовать Factory, Builder и Strategy вместе?**

Да. Они решают разные проблемы: Factory создаёт implementation, Builder собирает data, а Strategy предоставляет заменяемый behavior.

### Вопросы повышенного уровня

**1. Чем опасна большая Factory?**

Она может собрать несвязанные setup, configuration, logging и lifecycle logic. Это нарушает Single Responsibility Principle и делает изменения рискованными.

**2. Почему Builder defaults могут ослабить tests?**

Важные scenario data могут стать невидимыми. Читатель не поймёт, какие values влияют на result.

**3. Когда conditional лучше Strategy?**

Небольшой стабильный conditional лучше, если существует только один или два простых cases и не ожидается их рост.

**4. Почему один static `WebDriver` небезопасен для parallel tests?**

Tests используют общий mutable browser state. Один test может выполнить navigation, закрыть или изменить тот же driver, пока его использует другой test.

**5. Как решить, нужен ли паттерн?**

Нужно определить реальную проблему изменения или дублирования, сравнить паттерн с более простым решением и проверить, делает ли abstraction код понятнее и удобнее для тестирования.

### Вопросы с кодом

**1. Какая implementation будет возвращена?**

```java
WebDriver driver = BrowserFactory.create(Browser.FIREFOX);
```

**Ответ:** Simple Factory вернёт object `FirefoxDriver` через interface `WebDriver`.

**2. Какие значения содержит `admin`?**

```java
UserData admin = new UserDataBuilder()
    .withName("Ana")
    .withEmail("ana@example.com")
    .withRole("ADMIN")
    .build();
```

**Ответ:** Он содержит name `Ana`, email `ana@example.com`, role `ADMIN` и default value `active`, равное `true`.

**3. Что изменится при замене `BasicAuthStrategy` на `BearerTokenStrategy`?**

```java
AuthStrategy auth = new BearerTokenStrategy(token);
ApiClient client = new ApiClient(baseUri, auth);
```

**Ответ:** Изменится authentication behavior, но `ApiClient` и его request methods изменять не нужно.

**4. Почему этот design небезопасен для parallel tests?**

```java
public final class DriverStore {
    public static WebDriver driver;
}
```

**Ответ:** Все tests используют один mutable driver. Parallel tests могут перезаписать, изменить или закрыть одну browser session. Используй driver с test scope или thread scope.
