# Selenium Grid

## Table of Contents

- [[#Why Selenium Grid Matters]]
- [[#Local and Remote WebDriver]]
	- [[#RemoteWebDriver]]
- [[#How a Grid Session Is Created]]
- [[#Selenium Grid Components]]
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
- [[#Starting Selenium Grid]]
	- [[#Selenium Server JAR]]
	- [[#Docker Standalone]]
	- [[#Docker Compose Hub and Nodes]]
- [[#Options Capabilities and Session Matching]]
	- [[#Common Capabilities]]
	- [[#Grid Metadata]]
- [[#Parallel Execution and Capacity]]
	- [[#Grid vs Test Framework Parallelism]]
	- [[#One Driver per Test]]
	- [[#Capacity Planning]]
- [[#Remote Files]]
	- [[#File Upload]]
	- [[#File Download]]
- [[#Observability and Troubleshooting]]
	- [[#Grid UI and Status]]
	- [[#Logs Traces and Session ID]]
	- [[#Common Failure Causes]]
- [[#Security]]
- [[#Grid and Other Remote Platforms]]
- [[#Framework Integration]]
- [[#Common Mistakes]]
- [[#Interview Questions]]
	- [[#Beginner Questions]]
	- [[#Intermediate Questions]]
	- [[#Advanced Questions]]
	- [[#Code Questions]]

**Related notes:** [[AQA Java eng]]

> [!caution] Not in the video course
> Zaur's courses end at Java Core — Selenium Grid is not covered. This is the AQA layer: learn from this note and practice with a small local Grid before working with distributed infrastructure.

---

## Why Selenium Grid Matters

**Selenium Grid** routes WebDriver commands from test clients to browsers running on remote machines.

**Goal:** run browser tests on several browser, version, operating-system, and machine combinations without installing every environment on the test runner.

Grid is useful for:

- parallel execution on several machines
- cross-browser testing
- cross-platform testing
- running browser tests in CI
- sharing browser capacity between test runners
- reproducing problems on a specific browser environment

Grid does not make one test faster. It can make the whole suite finish sooner by running independent tests at the same time.

> [!info] Grid Is Infrastructure
> Selenium Grid provides remote browser sessions and routes commands. JUnit or TestNG still discovers tests, creates parallel work, and controls the test lifecycle.

---

## Local and Remote WebDriver

With a local driver, the test process starts a browser on the same machine. With a remote driver, the test sends WebDriver commands to a Selenium server that owns the browser session.

| Point | Local WebDriver | Remote WebDriver |
|---|---|---|
| Typical class | `ChromeDriver` | `RemoteWebDriver` |
| Browser location | Test machine | Grid Node or cloud machine |
| Driver service | Started locally | Managed by the remote environment |
| Required input | Browser `Options` are optional | Grid URL and browser `Options` are required |
| Main use | Development and debugging | CI, parallel, cross-browser, cross-platform |

### RemoteWebDriver

`RemoteWebDriver` implements the `WebDriver` interface. Most page and test code can use the same `WebDriver` type for local and remote execution.

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

The Java test runs on the client machine. The browser and browser driver run on a Grid Node.

> [!warning] Always End the Session
> Call `quit()` in teardown or `finally`. An abandoned remote session keeps a Grid slot and may block other tests until a timeout removes it.

---

## How a Grid Session Is Created

When the test creates `RemoteWebDriver`, Grid processes a new-session request:

1. The client sends the Grid URL and requested capabilities to the **Router**.
2. The Router places the request in the **New Session Queue**.
3. The **Distributor** looks for a free slot whose stereotype matches the capabilities.
4. The selected **Node** starts the browser session.
5. The Distributor stores the session ID and Node address in the **Session Map**.
6. Grid returns the created session to the client.
7. Later commands go through the Router to the Node that owns the session.
8. `quit()` ends the session and releases the slot.

If no matching slot is free, the request waits in the queue until capacity becomes available or the request times out.

---

## Selenium Grid Components

Selenium Grid 4 separates responsibilities into six components.

### Router

**Definition:** the Router is the external entry point of Grid.

**Goal:** forward each incoming request to the correct internal component.

- New-session requests go to the New Session Queue.
- Commands for an existing session go to the Node found through the Session Map.

### New Session Queue

**Definition:** the New Session Queue stores session requests that have not been assigned to a Node.

**Goal:** keep requests in FIFO order while Grid waits for a matching free slot.

A request can be rejected when its queue timeout is reached.

### Distributor

**Definition:** the Distributor tracks registered Nodes and assigns new sessions.

**Goal:** match requested capabilities with a free Node slot.

The Distributor:

- registers and monitors Nodes
- knows Node capabilities and available slots
- reads requests from the queue
- asks a matching Node to create a session
- updates the Session Map after creation

### Session Map

**Definition:** the Session Map stores the relation between a session ID and the Node that owns the session.

**Goal:** let the Router send each later WebDriver command to the correct Node.

### Node

**Definition:** a Node owns browser slots and runs WebDriver sessions.

**Goal:** execute browser commands and return results.

One Node can offer different browser types or versions. Each available execution place is a **slot**. Each slot has a **stereotype**: the minimum capabilities that a new session must match.

### Event Bus

**Definition:** the Event Bus carries asynchronous messages between Grid components.

**Goal:** support events such as Node registration without coupling every component through synchronous HTTP calls.

> [!tip] Component Responsibilities
> | Component | Main responsibility |
> |---|---|
> | Router | Accept and route external requests |
> | New Session Queue | Hold waiting session requests |
> | Distributor | Match requests to free slots |
> | Session Map | Map session IDs to Nodes |
> | Node | Run browser sessions |
> | Event Bus | Carry internal asynchronous events |

---

## Deployment Modes

The same six components can be packaged in different ways.

### Standalone

**Definition:** all Grid components run in one process on one machine.

**Advantages:**

- easiest setup
- useful for learning and local debugging
- convenient for small CI jobs

**Disadvantages:**

- limited to one machine
- one process is a single failure point
- limited scaling

### Hub and Node

**Definition:** the Hub contains Router, Distributor, Session Map, New Session Queue, and Event Bus; separate Nodes provide browser capacity.

**Advantages:**

- one entry point for different machines
- Nodes can use different operating systems and browsers
- capacity can be increased by adding Nodes

**Disadvantages:**

- requires network and port configuration
- the Hub must have enough resources
- operations are more complex than Standalone

### Distributed

**Definition:** every Grid component runs separately, often on different machines.

**Advantages:**

- components can scale independently
- suitable for large Grid installations
- failures can be isolated more precisely

**Disadvantages:**

- highest configuration and operational cost
- requires careful networking, monitoring, and startup configuration
- unnecessary for small teams

> [!tip] Deployment Mode Comparison
> | Mode | Machines | Complexity | Typical use |
> |---|---:|---:|---|
> | Standalone | 1 | Low | Learning, debugging, small CI |
> | Hub and Node | Several | Medium | Team Grid with several environments |
> | Distributed | Several | High | Large infrastructure with independent scaling |

---

## Starting Selenium Grid

### Selenium Server JAR

Prerequisites for a basic local Grid:

- Java 11 or newer
- Selenium Server JAR
- installed browsers
- available browser drivers, or Selenium Manager enabled

Start Standalone mode:

```bash
java -jar selenium-server-<version>.jar standalone
```

Start a Hub:

```bash
java -jar selenium-server-<version>.jar hub
```

Register a Node when the Hub uses default ports:

```bash
java -jar selenium-server-<version>.jar node --hub http://<hub-ip>:4444
```

When Hub and Node run on different machines, the machines must reach each other. The Hub Event Bus uses ports `4442` and `4443` by default, and the Node port must also be reachable.

### Docker Standalone

Docker provides an isolated browser, driver, and Grid server in one image.

```bash
docker run --rm -d \
  --name selenium-chrome \
  -p 4444:4444 \
  -p 7900:7900 \
  --shm-size="2g" \
  selenium/standalone-chrome:4.41.0-20260222
```

- Grid UI and WebDriver endpoint: `http://localhost:4444`
- noVNC browser view: `http://localhost:7900`
- `--shm-size` gives the browser more shared memory

Pin the image tag so local and CI environments use the same browser and Grid versions.

### Docker Compose Hub and Nodes

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

Start it:

```bash
docker compose up -d
```

Stop it:

```bash
docker compose down
```

The Docker concepts are covered in [[22 Docker and Test Environments]].

---

## Options Capabilities and Session Matching

**Capabilities** describe the requested browser session. Browser `Options` classes are the preferred way to build them.

```java
ChromeOptions options = new ChromeOptions();
options.setPlatformName("linux");
options.setAcceptInsecureCerts(true);
options.addArguments("--headless=new");

WebDriver driver = new RemoteWebDriver(gridUrl, options);
```

Grid compares the requested capabilities with Node slot stereotypes. A session can start only when a compatible slot exists.

### Common Capabilities

| Capability | Purpose |
|---|---|
| `browserName` | Requested browser; normally set by `ChromeOptions`, `FirefoxOptions`, and similar classes |
| `browserVersion` | Requested browser version |
| `platformName` | Requested operating-system platform |
| `acceptInsecureCerts` | Whether the browser accepts insecure TLS certificates |
| Browser-specific options | Arguments, preferences, profiles, and vendor settings |

Do not request a version or platform that no Node offers. Such a request waits in the queue and eventually times out.

### Grid Metadata

Grid-specific metadata uses the `se:` prefix.

```java
ChromeOptions options = new ChromeOptions();
options.setCapability("se:name", "user can log in");
options.setCapability("se:build", "regression-2026-07-22");
```

`se:name` makes the session easier to identify in Grid UI. Other metadata can help connect a session to a CI build or test report.

> [!caution] Capabilities Are Selection Rules
> More capabilities do not make a session better. Every strict capability reduces the number of matching slots and can increase queue time.

---

## Parallel Execution and Capacity

### Grid vs Test Framework Parallelism

Grid does not automatically run test methods in parallel. The test runner must create concurrent tests.

For JUnit 5, parallel execution can be enabled in `junit-platform.properties`:

```properties
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
```

If the runner starts 10 tests but Grid has 4 compatible free slots:

- 4 sessions can start
- the remaining requests wait in the New Session Queue
- waiting requests start as slots become free or fail after timeout

### One Driver per Test

Each parallel test needs its own WebDriver session.

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

A global static driver is unsafe because parallel tests can navigate or close the same session.

### Capacity Planning

Capacity depends on:

- available Node slots
- CPU and RAM
- browser type and version
- page complexity
- video, VNC, tracing, and logging overhead
- session creation rate
- network speed and stability

The Selenium documentation uses roughly one CPU and 1 GB RAM per browser session as a starting reference, but real projects must measure their own usage.

Prefer several small isolated Nodes over one very large Node when possible. A failed small Node affects fewer sessions.

---

## Remote Files

The test client and remote browser use different file systems. A path that exists on the test runner may not exist on the Node.

### File Upload

Use `LocalFileDetector` when a remote session must upload a local client file:

```java
RemoteWebDriver driver = new RemoteWebDriver(gridUrl, new ChromeOptions());
driver.setFileDetector(new LocalFileDetector());

WebElement fileInput = driver.findElement(By.cssSelector("input[type='file']"));
fileInput.sendKeys(Path.of("src/test/resources/avatar.png").toAbsolutePath().toString());
```

The file detector transfers the file from the client to the remote environment before the browser uses it.

### File Download

Remote downloads are saved on the Node, not automatically on the test client. Supported Grid and browser configurations can expose downloadable files through Selenium APIs.

```java
ChromeOptions options = new ChromeOptions();
options.setEnableDownloads(true);

RemoteWebDriver driver = new RemoteWebDriver(gridUrl, options);
```

For project-specific downloads, decide where files live, how tests retrieve them, and how cleanup works. Do not assume a local `Downloads` folder.

---

## Observability and Troubleshooting

### Grid UI and Status

Useful endpoints:

| Endpoint | Purpose |
|---|---|
| `http://localhost:4444` | Grid UI with Nodes, slots, and sessions |
| `http://localhost:4444/status` | Readiness and Grid status as JSON |
| `http://localhost:4444/graphql` | Query detailed Grid information |

Check Grid readiness before starting a large CI suite.

### Logs Traces and Session ID

Useful diagnostic data includes:

- test name and CI build ID
- Grid session ID
- requested and actual capabilities
- Node address
- browser and driver versions
- queue time and session creation time
- client, Grid, Node, driver, and browser logs
- screenshots, video, console logs, and network data when available

Get the session ID in Java:

```java
RemoteWebDriver remote = (RemoteWebDriver) driver;
String sessionId = remote.getSessionId().toString();
```

Grid 4 supports structured logs and distributed tracing. Traces help follow one request across multiple Grid components.

### Common Failure Causes

| Symptom | Likely cause | What to check |
|---|---|---|
| New session waits and times out | No matching free slot | Capabilities, Node stereotypes, capacity, queue timeout |
| Node does not appear in UI | Registration or network problem | Hub address, Event Bus ports, Node port, firewall, logs |
| `SessionNotCreatedException` | Browser or driver problem, bad capability, low resources | Actual versions, Node logs, disk, memory, shared memory |
| Test works locally but fails remotely | Environment difference | Files, fonts, timezone, locale, window size, headless mode |
| Commands fail during a session | Node or network became unavailable | Session ID, Node health, Grid logs, network errors |
| Sessions remain busy | Missing or failed cleanup | `quit()` in teardown, test process termination, session timeout |
| Parallel tests affect one another | Shared mutable test state | Static driver, shared users, shared files, shared environment data |

---

## Security

Selenium Grid can control browsers that may access internal applications and files. Do not expose Grid directly to the public internet.

Protect it with:

- firewall and private network rules
- authentication or a protected reverse proxy
- TLS where traffic crosses untrusted networks
- restricted access to Grid UI and APIs
- secret masking in capabilities, logs, screenshots, and video
- patched Grid, browser, driver, image, and operating-system versions
- isolated Node permissions and containers

> [!danger] Never Publish an Open Grid
> An unprotected Grid can give other people access to browser infrastructure, internal websites, files, and executable environments.

---

## Grid and Other Remote Platforms

| Platform | Infrastructure owner | Main idea |
|---|---|---|
| Selenium Grid | Your team | Official Selenium remote execution infrastructure |
| Docker Selenium | Your team | Selenium Grid packaged in maintained browser containers |
| Cloud browser service | Provider | Managed browsers, platforms, scaling, video, and reports |
| Selenoid | Your team | Alternative Selenium-compatible service that commonly starts browser containers per session |

All these options can accept remote WebDriver sessions, but their capabilities, artifacts, scaling, security, and cost differ.

Choose based on:

- required browsers and operating systems
- parallel capacity
- maintenance skills and time
- network and security requirements
- logs, video, and debugging features
- budget

---

## Framework Integration

Keep local and remote creation behind one driver factory:

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

Read the execution mode and Grid URL from configuration:

```java
ExecutionMode mode = ExecutionMode.valueOf(
    System.getProperty("execution.mode", "LOCAL").toUpperCase()
);

URL gridUrl = URI.create(
    System.getProperty("grid.url", "http://localhost:4444")
).toURL();
```

Tests should depend on `WebDriver`, not on `ChromeDriver` or `RemoteWebDriver`. Driver lifecycle must stay at test or thread scope.

This uses Factory ideas from [[24 Design Patterns for AQA]] and architecture principles from [[20 Automation Framework Architecture]].

---

## Common Mistakes

| Mistake | Why it is a problem | Better approach |
|---|---|---|
| Expecting Grid to create parallel tests | Grid only provides remote slots | Enable parallelism in JUnit or TestNG |
| Using one static driver for all threads | Tests share and close the same session | Create one driver per test or thread |
| Requesting unavailable capabilities | The session stays in the queue | Match requests to Node stereotypes |
| Running more sessions than hardware supports | Browsers become slow or crash | Measure resource use and limit capacity |
| Forgetting `quit()` | Slots remain busy | Close sessions in guaranteed teardown |
| Using client file paths on the Node | The remote machine cannot see the file | Use `LocalFileDetector` or shared storage |
| Using unpinned Docker image tags | Browser versions change unexpectedly | Pin and update versions intentionally |
| Logging without session IDs | Client and Grid events are hard to connect | Record session ID and capabilities |
| Treating Grid errors as test failures | Infrastructure noise hides product quality | Classify application and infrastructure failures separately |
| Exposing port 4444 publicly | Other people may control the infrastructure | Use private networking, firewall, and authentication |
| Starting with Distributed mode | Operations become complex too early | Begin with Standalone or Hub and Node |

---

## Interview Questions

### Beginner Questions

**1. What is Selenium Grid?**

Selenium Grid routes WebDriver commands to browser sessions running on remote machines.

**2. Why is Selenium Grid used?**

It supports parallel, cross-browser, and cross-platform test execution on shared remote infrastructure.

**3. What is `RemoteWebDriver`?**

It is a `WebDriver` implementation that sends commands to a remote Selenium server using a Grid URL and browser `Options`.

**4. What is a Grid Node?**

A Node provides browser slots and runs WebDriver sessions.

**5. Does Grid automatically run JUnit tests in parallel?**

No. JUnit or TestNG creates parallel tests. Grid provides the remote capacity for their browser sessions.

**6. What is the default Grid URL for a local setup?**

It is usually `http://localhost:4444`.

### Intermediate Questions

**1. What happens when a new remote session is requested?**

The Router sends it to the queue, the Distributor finds a matching free slot, a Node creates the browser, and the Session Map stores the session-to-Node relation.

**2. What is the difference between Standalone and Hub–Node modes?**

Standalone runs all components in one process on one machine. Hub–Node separates the central components from browser Nodes and can use several machines.

**3. How does Grid select a Node?**

The Distributor compares requested capabilities with free slot stereotypes and selects a compatible slot.

**4. Why can a session stay in the queue?**

All compatible slots may be busy, or no Node may offer the requested browser, version, or platform.

**5. Why is `LocalFileDetector` needed?**

It transfers a client-side file to the remote environment before a remote browser uploads it.

**6. What information is useful when diagnosing a Grid failure?**

Session ID, requested and actual capabilities, Node address, queue time, browser versions, and client, Grid, Node, driver, and browser logs are useful.

### Advanced Questions

**1. What are the six main Selenium Grid 4 components?**

Router, New Session Queue, Distributor, Session Map, Node, and Event Bus.

**2. What is the difference between a slot and a stereotype?**

A slot is a place where one session can run. Its stereotype is the minimum capability set that a request must match.

**3. Why can increasing Node session limits make tests slower?**

Too many browsers compete for CPU, memory, disk, and network resources. More slots do not create more hardware capacity.

**4. Why should application failures and Grid failures be classified separately?**

An infrastructure failure does not prove a product defect. Separate classification gives correct quality metrics and retry decisions.

**5. When is Distributed mode appropriate?**

It is appropriate for large installations that need independent component scaling and have the operational ability to manage networking, monitoring, and failures.

**6. Why is a public Selenium Grid dangerous?**

It can allow unauthorized users to control browsers, reach internal applications, access files, or run code in the infrastructure.

### Code Questions

**1. Where does the browser run?**

```java
WebDriver driver = new RemoteWebDriver(
    URI.create("http://grid:4444").toURL(),
    new ChromeOptions()
);
```

**Answer:** The Java test runs on the client machine, while Grid creates the Chrome session on a matching remote Node.

**2. What happens if Grid has no free Linux Chrome slot?**

```java
ChromeOptions options = new ChromeOptions();
options.setPlatformName("linux");

WebDriver driver = new RemoteWebDriver(gridUrl, options);
```

**Answer:** The request waits in the New Session Queue. It starts when a matching slot becomes free or fails when the request timeout is reached.

**3. Why is this code unsafe with concurrent tests?**

```java
public final class DriverStore {
    public static WebDriver driver;
}
```

**Answer:** Every test thread shares the same mutable session. One test can navigate or close the browser while another test is using it.

**4. Why can this upload fail with Grid?**

```java
driver.findElement(By.id("file"))
    .sendKeys("C:\\tests\\avatar.png");
```

**Answer:** The path exists on the client but may not exist on the remote Node. Configure `LocalFileDetector` or use a file location available to the remote environment.
