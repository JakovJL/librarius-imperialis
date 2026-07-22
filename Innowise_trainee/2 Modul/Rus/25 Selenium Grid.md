# Selenium Grid

## Содержание

- [[#Зачем нужен Selenium Grid]]
- [[#Local и Remote WebDriver]]
	- [[#RemoteWebDriver]]
- [[#Как создаётся Grid Session]]
- [[#Компоненты Selenium Grid]]
	- [[#Router]]
	- [[#New Session Queue]]
	- [[#Distributor]]
	- [[#Session Map]]
	- [[#Node]]
	- [[#Event Bus]]
- [[#Deployment Modes]]
	- [[#Standalone]]
	- [[#Hub and Node]]
	- [[#Distributed]]
- [[#Запуск Selenium Grid]]
	- [[#Selenium Server JAR]]
	- [[#Docker Standalone]]
	- [[#Docker Compose с Hub и Nodes]]
- [[#Options Capabilities и Session Matching]]
	- [[#Основные Capabilities]]
	- [[#Grid Metadata]]
- [[#Parallel Execution и Capacity]]
	- [[#Grid и параллельность Test Framework]]
	- [[#Один Driver на Test]]
	- [[#Capacity Planning]]
- [[#Remote Files]]
	- [[#File Upload]]
	- [[#File Download]]
- [[#Observability и диагностика]]
	- [[#Grid UI и Status]]
	- [[#Logs Traces и Session ID]]
	- [[#Типичные причины падений]]
- [[#Security]]
- [[#Grid и другие Remote Platforms]]
- [[#Интеграция во Framework]]
- [[#Типичные ошибки]]
- [[#Вопросы для собеседования]]
	- [[#Вопросы начального уровня]]
	- [[#Вопросы среднего уровня]]
	- [[#Вопросы повышенного уровня]]
	- [[#Вопросы с кодом]]

**Связанные заметки:** [[AQA Java rus]]

> [!caution] Нет в видео-курсе
> Курсы Заура заканчиваются на Java Core — Selenium Grid не разбирается. Это AQA-надстройка: учись по этой заметке и сначала практикуйся с небольшим local Grid, а затем переходи к distributed infrastructure.

---

## Зачем нужен Selenium Grid

**Selenium Grid** направляет WebDriver commands от test clients к browsers, запущенным на remote machines.

**Цель:** запускать browser tests в разных browsers, versions, operating systems и machines без установки всех environments на test runner.

Grid полезен для:

- parallel execution на нескольких machines
- cross-browser testing
- cross-platform testing
- запуска browser tests в CI
- общего использования browser capacity разными test runners
- воспроизведения проблемы в конкретном browser environment

Grid не ускоряет один test. Он может сократить время всего test suite за счёт одновременного запуска независимых tests.

> [!info] Grid — это инфраструктура
> Selenium Grid предоставляет remote browser sessions и направляет commands. JUnit или TestNG по-прежнему находит tests, создаёт parallel work и управляет test lifecycle.

---

## Local и Remote WebDriver

При использовании local driver test process запускает browser на той же machine. При использовании remote driver test отправляет WebDriver commands на Selenium server, который управляет browser session.

| Пункт | Local WebDriver | Remote WebDriver |
|---|---|---|
| Типичный class | `ChromeDriver` | `RemoteWebDriver` |
| Где работает browser | На test machine | На Grid Node или cloud machine |
| Driver service | Запускается локально | Управляется remote environment |
| Обязательные данные | Browser `Options` необязательны | Нужны Grid URL и browser `Options` |
| Основное применение | Development и debugging | CI, parallel, cross-browser и cross-platform testing |

### RemoteWebDriver

`RemoteWebDriver` реализует interface `WebDriver`. Большинство pages и tests могут использовать один type `WebDriver` для local и remote execution.

```java
import java.net.URI;
import java.net.URL;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.RemoteWebDriver;

URL gridUrl = URI.create("http://localhost:4444").toURL();

ChromeOptions options = new ChromeOptions();
options.addArguments("--window-size=1920,1080");

WebDriver driver = new RemoteWebDriver(gridUrl, options);

try {
    driver.get("https://example.com");
    System.out.println(driver.getTitle());
} finally {
    driver.quit();
}
```

Java test работает на client machine. Browser и browser driver работают на Grid Node.

> [!warning] Всегда завершай Session
> Вызывай `quit()` в teardown или `finally`. Незавершённая remote session занимает Grid slot и может блокировать другие tests, пока timeout не удалит её.

---

## Как создаётся Grid Session

Когда test создаёт `RemoteWebDriver`, Grid обрабатывает new session request:

1. Client отправляет Grid URL и requested capabilities в **Router**.
2. Router помещает request в **New Session Queue**.
3. **Distributor** ищет свободный slot, stereotype которого соответствует capabilities.
4. Выбранный **Node** запускает browser session.
5. Distributor сохраняет session ID и адрес Node в **Session Map**.
6. Grid возвращает созданную session клиенту.
7. Следующие commands проходят через Router к Node, который владеет session.
8. `quit()` завершает session и освобождает slot.

Если подходящего свободного slot нет, request ждёт в queue до освобождения capacity или до истечения timeout.

---

## Компоненты Selenium Grid

Selenium Grid 4 разделяет responsibilities между шестью components.

### Router

**Определение:** Router — внешняя entry point Grid.

**Цель:** направлять каждый входящий request в правильный внутренний component.

- New session requests отправляются в New Session Queue.
- Commands существующей session отправляются на Node, найденный через Session Map.

### New Session Queue

**Определение:** New Session Queue хранит session requests, которые ещё не назначены Node.

**Цель:** сохранять requests в порядке FIFO, пока Grid ждёт подходящий свободный slot.

Request может быть отклонён после истечения queue timeout.

### Distributor

**Определение:** Distributor отслеживает зарегистрированные Nodes и назначает новые sessions.

**Цель:** сопоставлять requested capabilities со свободным Node slot.

Distributor:

- регистрирует и проверяет Nodes
- знает capabilities Nodes и доступные slots
- читает requests из queue
- просит подходящий Node создать session
- обновляет Session Map после создания

### Session Map

**Определение:** Session Map хранит связь между session ID и Node, который владеет session.

**Цель:** позволить Router отправлять каждый следующий WebDriver command на правильный Node.

### Node

**Определение:** Node управляет browser slots и запускает WebDriver sessions.

**Цель:** выполнять browser commands и возвращать results.

Один Node может предоставлять разные browser types или versions. Каждое доступное место выполнения называется **slot**. Каждый slot имеет **stereotype** — минимальный набор capabilities, которому должна соответствовать new session.

### Event Bus

**Определение:** Event Bus передаёт asynchronous messages между Grid components.

**Цель:** поддерживать events, например регистрацию Node, без связи всех components только через synchronous HTTP calls.

> [!tip] Обязанности компонентов
> | Компонент | Основная обязанность |
> |---|---|
> | Router | Принимать и направлять внешние requests |
> | New Session Queue | Хранить ожидающие session requests |
> | Distributor | Сопоставлять requests со свободными slots |
> | Session Map | Связывать session IDs с Nodes |
> | Node | Запускать browser sessions |
> | Event Bus | Передавать внутренние asynchronous events |

---

## Deployment Modes

Одинаковые шесть components можно объединять разными способами.

### Standalone

**Определение:** все Grid components работают в одном process на одной machine.

**Преимущества:**

- самая простая настройка
- подходит для обучения и local debugging
- удобен для небольших CI jobs

**Недостатки:**

- ограничен одной machine
- один process является единой failure point
- ограниченное scaling

### Hub and Node

**Определение:** Hub содержит Router, Distributor, Session Map, New Session Queue и Event Bus, а отдельные Nodes предоставляют browser capacity.

**Преимущества:**

- одна entry point для разных machines
- Nodes могут использовать разные operating systems и browsers
- capacity можно увеличить добавлением Nodes

**Недостатки:**

- нужна настройка network и ports
- Hub должен иметь достаточно resources
- эксплуатация сложнее, чем Standalone

### Distributed

**Определение:** каждый Grid component работает отдельно, часто на разных machines.

**Преимущества:**

- components можно масштабировать независимо
- подходит для больших Grid installations
- failures можно точнее изолировать

**Недостатки:**

- самая высокая стоимость настройки и эксплуатации
- требует аккуратной настройки networking, monitoring и startup
- не нужен небольшим teams

> [!tip] Сравнение Deployment Modes
> | Mode | Machines | Сложность | Обычное применение |
> |---|---:|---:|---|
> | Standalone | 1 | Низкая | Обучение, debugging, небольшой CI |
> | Hub and Node | Несколько | Средняя | Team Grid с несколькими environments |
> | Distributed | Несколько | Высокая | Крупная infrastructure с независимым scaling |

---

## Запуск Selenium Grid

### Selenium Server JAR

Prerequisites для простого local Grid:

- Java 11 или новее
- Selenium Server JAR
- установленные browsers
- доступные browser drivers или включённый Selenium Manager

Запуск Standalone mode:

```bash
java -jar selenium-server-<version>.jar standalone
```

Запуск Hub:

```bash
java -jar selenium-server-<version>.jar hub
```

Регистрация Node, если Hub использует default ports:

```bash
java -jar selenium-server-<version>.jar node --hub http://<hub-ip>:4444
```

Если Hub и Node работают на разных machines, они должны видеть друг друга по network. Event Bus Hub по умолчанию использует ports `4442` и `4443`; port Node также должен быть доступен.

### Docker Standalone

Docker предоставляет изолированные browser, driver и Grid server в одном image.

```bash
docker run --rm -d \
  --name selenium-chrome \
  -p 4444:4444 \
  -p 7900:7900 \
  --shm-size="2g" \
  selenium/standalone-chrome:4.41.0-20260222
```

- Grid UI и WebDriver endpoint: `http://localhost:4444`
- noVNC browser view: `http://localhost:7900`
- `--shm-size` предоставляет browser больше shared memory

Закрепляй image tag, чтобы local и CI environments использовали одинаковые versions browser и Grid.

### Docker Compose с Hub и Nodes

```yaml
services:
  selenium-hub:
    image: selenium/hub:4.41.0-20260222
    container_name: selenium-hub
    ports:
      - "4444:4444"

  chrome:
    image: selenium/node-chrome:4.41.0-20260222
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443

  firefox:
    image: selenium/node-firefox:4.41.0-20260222
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
```

Запуск:

```bash
docker compose up -d
```

Остановка:

```bash
docker compose down
```

Основы Docker рассматриваются в [[22 Docker и тестовые окружения]].

---

## Options Capabilities и Session Matching

**Capabilities** описывают requested browser session. Предпочтительный способ их создания — browser `Options` classes.

```java
ChromeOptions options = new ChromeOptions();
options.setPlatformName("linux");
options.setAcceptInsecureCerts(true);
options.addArguments("--headless=new");

WebDriver driver = new RemoteWebDriver(gridUrl, options);
```

Grid сравнивает requested capabilities со stereotypes Node slots. Session может запуститься только при наличии совместимого slot.

### Основные Capabilities

| Capability | Назначение |
|---|---|
| `browserName` | Requested browser; обычно задаётся через `ChromeOptions`, `FirefoxOptions` и похожие classes |
| `browserVersion` | Requested browser version |
| `platformName` | Requested operating-system platform |
| `acceptInsecureCerts` | Разрешает browser принимать недоверенные TLS certificates |
| Browser-specific options | Arguments, preferences, profiles и vendor settings |

Не запрашивай version или platform, которые не предоставляет ни один Node. Такой request будет ждать в queue и завершится по timeout.

### Grid Metadata

Grid-specific metadata использует prefix `se:`.

```java
ChromeOptions options = new ChromeOptions();
options.setCapability("se:name", "user can log in");
options.setCapability("se:build", "regression-2026-07-22");
```

`se:name` помогает найти session в Grid UI. Другие metadata помогают связать session с CI build или test report.

> [!caution] Capabilities — это правила выбора
> Большее количество capabilities не делает session лучше. Каждая строгая capability уменьшает число подходящих slots и может увеличить queue time.

---

## Parallel Execution и Capacity

### Grid и параллельность Test Framework

Grid не запускает test methods параллельно автоматически. Test runner должен создать concurrent tests.

Для JUnit 5 parallel execution можно включить в `junit-platform.properties`:

```properties
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
```

Если runner запускает 10 tests, а Grid имеет 4 подходящих свободных slots:

- 4 sessions могут запуститься
- остальные requests будут ждать в New Session Queue
- ожидающие requests запустятся после освобождения slots или упадут по timeout

### Один Driver на Test

Каждому parallel test нужна отдельная WebDriver session.

```java
class LoginTest {
    private WebDriver driver;

    @BeforeEach
    void setUp() throws Exception {
        URL gridUrl = URI.create("http://localhost:4444").toURL();
        driver = new RemoteWebDriver(gridUrl, new ChromeOptions());
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

Global static driver небезопасен: parallel tests могут выполнять navigation или закрывать одну session.

### Capacity Planning

Capacity зависит от:

- доступных Node slots
- CPU и RAM
- browser type и version
- сложности страниц
- overhead от video, VNC, tracing и logging
- скорости создания sessions
- скорости и стабильности network

Документация Selenium использует примерно один CPU и 1 GB RAM на одну browser session как начальную оценку, но реальный project должен измерить собственное потребление.

По возможности используй несколько небольших изолированных Nodes вместо одного очень большого Node. Падение небольшого Node затронет меньше sessions.

---

## Remote Files

Test client и remote browser используют разные file systems. Path, существующий на test runner, может отсутствовать на Node.

### File Upload

Используй `LocalFileDetector`, если remote session должна загрузить local client file:

```java
RemoteWebDriver driver = new RemoteWebDriver(gridUrl, new ChromeOptions());
driver.setFileDetector(new LocalFileDetector());

WebElement fileInput = driver.findElement(By.cssSelector("input[type='file']"));
fileInput.sendKeys(Path.of("src/test/resources/avatar.png").toAbsolutePath().toString());
```

File detector переносит file с client в remote environment до того, как browser начнёт его использовать.

### File Download

Remote downloads сохраняются на Node, а не автоматически на test client. Поддерживаемые Grid и browser configurations могут предоставлять downloadable files через Selenium APIs.

```java
ChromeOptions options = new ChromeOptions();
options.setEnableDownloads(true);

RemoteWebDriver driver = new RemoteWebDriver(gridUrl, options);
```

Для project-specific downloads определи, где находятся files, как tests получают их и как выполняется cleanup. Не рассчитывай на local папку `Downloads`.

---

## Observability и диагностика

### Grid UI и Status

Полезные endpoints:

| Endpoint | Назначение |
|---|---|
| `http://localhost:4444` | Grid UI с Nodes, slots и sessions |
| `http://localhost:4444/status` | Readiness и Grid status в JSON |
| `http://localhost:4444/graphql` | Запрос подробной Grid information |

Проверяй Grid readiness до запуска большого CI suite.

### Logs Traces и Session ID

Полезные diagnostic data:

- test name и CI build ID
- Grid session ID
- requested и actual capabilities
- адрес Node
- browser и driver versions
- queue time и session creation time
- client, Grid, Node, driver и browser logs
- screenshots, video, console logs и network data, если они доступны

Получение session ID в Java:

```java
RemoteWebDriver remote = (RemoteWebDriver) driver;
String sessionId = remote.getSessionId().toString();
```

Grid 4 поддерживает structured logs и distributed tracing. Traces помогают проследить один request через несколько Grid components.

### Типичные причины падений

| Симптом | Возможная причина | Что проверить |
|---|---|---|
| New session ждёт и падает по timeout | Нет подходящего свободного slot | Capabilities, Node stereotypes, capacity, queue timeout |
| Node отсутствует в UI | Проблема registration или network | Hub address, Event Bus ports, Node port, firewall, logs |
| `SessionNotCreatedException` | Проблема browser или driver, плохая capability, нехватка resources | Actual versions, Node logs, disk, memory, shared memory |
| Test работает local, но падает remote | Различие environments | Files, fonts, timezone, locale, window size, headless mode |
| Commands падают во время session | Node или network стали недоступны | Session ID, Node health, Grid logs, network errors |
| Sessions остаются занятыми | Cleanup отсутствует или упал | `quit()` в teardown, завершение test process, session timeout |
| Parallel tests влияют друг на друга | Shared mutable test state | Static driver, shared users, shared files, shared environment data |

---

## Security

Selenium Grid управляет browsers, которые могут иметь доступ к internal applications и files. Не открывай Grid напрямую в public internet.

Защищай его с помощью:

- firewall и private network rules
- authentication или защищённого reverse proxy
- TLS, если traffic проходит через untrusted networks
- ограничения доступа к Grid UI и APIs
- маскирования secrets в capabilities, logs, screenshots и video
- обновлённых Grid, browser, driver, images и operating systems
- изолированных permissions и containers для Nodes

> [!danger] Не публикуй открытый Grid
> Незащищённый Grid может дать посторонним доступ к browser infrastructure, internal websites, files и executable environments.

---

## Grid и другие Remote Platforms

| Platform | Кто управляет infrastructure | Основная идея |
|---|---|---|
| Selenium Grid | Твоя team | Официальная Selenium infrastructure для remote execution |
| Docker Selenium | Твоя team | Selenium Grid в поддерживаемых browser containers |
| Cloud browser service | Provider | Управляемые browsers, platforms, scaling, video и reports |
| Selenoid | Твоя team | Альтернативный Selenium-compatible service, обычно запускающий browser containers для каждой session |

Все варианты могут принимать remote WebDriver sessions, но отличаются capabilities, artifacts, scaling, security и стоимостью.

Выбор зависит от:

- необходимых browsers и operating systems
- parallel capacity
- времени и навыков для maintenance
- network и security requirements
- logs, video и debugging features
- бюджета

---

## Интеграция во Framework

Скрой local и remote создание за одной driver factory:

```java
public enum ExecutionMode {
    LOCAL,
    GRID
}
```

```java
public final class DriverFactory {

    public WebDriver create(ExecutionMode mode, URL gridUrl) {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--window-size=1920,1080");

        return switch (mode) {
            case LOCAL -> new ChromeDriver(options);
            case GRID -> new RemoteWebDriver(gridUrl, options);
        };
    }
}
```

Читай execution mode и Grid URL из configuration:

```java
ExecutionMode mode = ExecutionMode.valueOf(
    System.getProperty("execution.mode", "LOCAL").toUpperCase()
);

URL gridUrl = URI.create(
    System.getProperty("grid.url", "http://localhost:4444")
).toURL();
```

Tests должны зависеть от `WebDriver`, а не от `ChromeDriver` или `RemoteWebDriver`. Driver lifecycle должен иметь test scope или thread scope.

Здесь используются идеи Factory из [[24 Паттерны проектирования для AQA]] и принципы architecture из [[20 Архитектура фреймворка автотестов]].

---

## Типичные ошибки

| Ошибка | Почему это проблема | Что делать лучше |
|---|---|---|
| Ожидать, что Grid сам создаст parallel tests | Grid предоставляет только remote slots | Включи parallelism в JUnit или TestNG |
| Использовать один static driver для всех threads | Tests используют и закрывают одну session | Создавай один driver на test или thread |
| Запрашивать недоступные capabilities | Session остаётся в queue | Сопоставляй requests со stereotypes Nodes |
| Запускать больше sessions, чем поддерживает hardware | Browsers замедляются или падают | Измеряй resource usage и ограничивай capacity |
| Забывать `quit()` | Slots остаются занятыми | Закрывай sessions в гарантированном teardown |
| Использовать client file paths на Node | Remote machine не видит file | Используй `LocalFileDetector` или shared storage |
| Использовать незакреплённые Docker image tags | Browser versions неожиданно меняются | Закрепляй и обновляй versions явно |
| Логировать без session IDs | Client и Grid events трудно связать | Сохраняй session ID и capabilities |
| Считать Grid errors падениями test | Infrastructure noise скрывает product quality | Отдельно классифицируй application и infrastructure failures |
| Открывать port 4444 в public network | Посторонние могут управлять infrastructure | Используй private network, firewall и authentication |
| Начинать с Distributed mode | Эксплуатация становится сложной слишком рано | Начни со Standalone или Hub and Node |

---

## Вопросы для собеседования

### Вопросы начального уровня

**1. Что такое Selenium Grid?**

Selenium Grid направляет WebDriver commands к browser sessions, запущенным на remote machines.

**2. Зачем используется Selenium Grid?**

Он поддерживает parallel, cross-browser и cross-platform test execution на общей remote infrastructure.

**3. Что такое `RemoteWebDriver`?**

Это implementation `WebDriver`, которая отправляет commands на remote Selenium server с использованием Grid URL и browser `Options`.

**4. Что такое Grid Node?**

Node предоставляет browser slots и запускает WebDriver sessions.

**5. Запускает ли Grid JUnit tests параллельно автоматически?**

Нет. Parallel tests создаёт JUnit или TestNG. Grid предоставляет remote capacity для их browser sessions.

**6. Какой default Grid URL используется при local setup?**

Обычно используется `http://localhost:4444`.

### Вопросы среднего уровня

**1. Что происходит при запросе новой remote session?**

Router отправляет request в queue, Distributor находит подходящий свободный slot, Node создаёт browser, а Session Map сохраняет связь между session и Node.

**2. В чём разница между Standalone и Hub–Node modes?**

Standalone запускает все components в одном process на одной machine. Hub–Node отделяет central components от browser Nodes и может использовать несколько machines.

**3. Как Grid выбирает Node?**

Distributor сравнивает requested capabilities со stereotypes свободных slots и выбирает совместимый slot.

**4. Почему session может остаться в queue?**

Все совместимые slots могут быть заняты, или ни один Node не предоставляет requested browser, version или platform.

**5. Зачем нужен `LocalFileDetector`?**

Он переносит client-side file в remote environment до того, как remote browser загрузит его.

**6. Какие данные полезны при диагностике Grid failure?**

Полезны session ID, requested и actual capabilities, адрес Node, queue time, browser versions и client, Grid, Node, driver и browser logs.

### Вопросы повышенного уровня

**1. Какие шесть основных components есть в Selenium Grid 4?**

Router, New Session Queue, Distributor, Session Map, Node и Event Bus.

**2. В чём разница между slot и stereotype?**

Slot — это место для запуска одной session. Его stereotype — минимальный набор capabilities, которому должен соответствовать request.

**3. Почему увеличение session limits Node может замедлить tests?**

Слишком много browsers конкурируют за CPU, memory, disk и network resources. Дополнительные slots не создают дополнительную hardware capacity.

**4. Почему application failures и Grid failures нужно классифицировать отдельно?**

Infrastructure failure не доказывает product defect. Отдельная классификация даёт правильные quality metrics и retry decisions.

**5. Когда подходит Distributed mode?**

Он подходит большим installations, которым нужно независимое scaling components и которые могут управлять networking, monitoring и failures.

**6. Чем опасен public Selenium Grid?**

Он может позволить посторонним управлять browsers, получать доступ к internal applications и files или запускать code в infrastructure.

### Вопросы с кодом

**1. Где будет работать browser?**

```java
WebDriver driver = new RemoteWebDriver(
    URI.create("http://grid:4444").toURL(),
    new ChromeOptions()
);
```

**Ответ:** Java test работает на client machine, а Grid создаёт Chrome session на подходящем remote Node.

**2. Что произойдёт, если в Grid нет свободного Linux Chrome slot?**

```java
ChromeOptions options = new ChromeOptions();
options.setPlatformName("linux");

WebDriver driver = new RemoteWebDriver(gridUrl, options);
```

**Ответ:** Request будет ждать в New Session Queue. Он запустится после освобождения подходящего slot или упадёт после истечения request timeout.

**3. Почему этот код небезопасен при concurrent tests?**

```java
public final class DriverStore {
    public static WebDriver driver;
}
```

**Ответ:** Все test threads используют одну mutable session. Один test может выполнить navigation или закрыть browser, пока его использует другой test.

**4. Почему file upload может упасть при работе через Grid?**

```java
driver.findElement(By.id("file"))
    .sendKeys("C:\\tests\\avatar.png");
```

**Ответ:** Path существует на client, но может отсутствовать на remote Node. Настрой `LocalFileDetector` или используй file location, доступный remote environment.
