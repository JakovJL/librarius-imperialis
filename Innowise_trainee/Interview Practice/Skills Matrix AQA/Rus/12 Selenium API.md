# Selenium API

## Содержание

- [[#Вопросы и ответы]]

**Связанные заметки:** [[00 Индекс Skills Matrix AQA]]

---

## Вопросы и ответы

1. Как узнать версию Selenium?

   **Ответ:** Надёжнее всего проверить объявленную dependency в `pom.xml`, `build.gradle` или lock file и подтвердить фактически разрешённую версию через dependency tree build tool. В Maven для этого можно использовать `mvn dependency:tree` и найти `org.seleniumhq.selenium`. Версия в manifest runtime-библиотеки иногда доступна через Java Package API, но она может отсутствовать, поэтому build configuration остаётся основным источником.

2. Как узнать версию драйвера?

   **Ответ:** Для локального binary можно выполнить команду вроде `chromedriver --version`, `geckodriver --version` или посмотреть debug logs Selenium Manager. Browser version доступна в фактических session capabilities, а версия driver иногда передаётся в browser-specific capability, но это не переносимый способ для всех браузеров. В remote environment версию лучше также проверять по конфигурации и логам node.

3. Основные методы класса WebDriver

   **Ответ:** `get()` и `navigate()` открывают страницы, `getCurrentUrl()`, `getTitle()` и `getPageSource()` читают состояние документа. `findElement()` и `findElements()` ищут элементы, `switchTo()` меняет context на frame, window или alert, а `manage()` открывает cookies, timeouts, window и logs. `close()` закрывает текущую вкладку или окно, а `quit()` завершает всю session и процессы driver.

4. Основные методы класса WebElement

   **Ответ:** Для действий используются `click()`, `sendKeys()`, `clear()` и `submit()`, а для чтения — `getText()`, `getAttribute()`, `getDomAttribute()`, `getDomProperty()` и `getCssValue()`. Состояние проверяют через `isDisplayed()`, `isEnabled()` и `isSelected()`. Элемент также поддерживает относительный `findElement()`, получение tag name, position и size, но сохранённая ссылка может стать stale после изменения DOM.

5. Основные методы класса By

   **Ответ:** `By` создаёт locator strategy через статические методы `id()`, `name()`, `className()`, `tagName()`, `linkText()`, `partialLinkText()`, `cssSelector()` и `xpath()`. Затем locator передаётся в `findElement()` или `findElements()`. Я предпочитаю короткий уникальный locator по стабильному id или `data-*`, а сложный XPath использую только при реальной необходимости.

6. Методы класса WebDriver, WebElement, By

   **Ответ:** Эти типы отвечают за разные уровни API: WebDriver управляет browser session и context, WebElement представляет конкретный найденный DOM element, а By только описывает способ поиска. Например, `driver.findElement(By.id("login"))` возвращает WebElement, после чего можно вызвать `click()` или `sendKeys()`. Такое разделение позволяет хранить locator до момента поиска и повторно находить элемент после изменения DOM.

7. Actions

   **Ответ:** Класс `Actions` строит сложную последовательность пользовательских input actions: движение мыши, hover, click, double click, context click, drag and drop, клавиши и scrolling. Вызовы образуют builder chain, а `perform()` выполняет собранные действия. Actions нужен для реалистичных составных жестов, но обычный `WebElement.click()` проще и предпочтительнее для стандартного клика.

8. Expected Conditions

   **Ответ:** Expected Conditions — готовые условия для Explicit Wait, например presence, visibility, clickability, изменение URL, наличие alert или число windows. `WebDriverWait` опрашивает условие до успеха или timeout и возвращает значение либо выбрасывает `TimeoutException`. Условие нужно выбирать по требуемому состоянию, потому что присутствующий в DOM element ещё не обязательно видим или доступен для клика.

9. ExecuteScript (JSExecutor)

   **Ответ:** В Java driver приводится к `JavascriptExecutor`, после чего `executeScript()` выполняет JavaScript в context текущего window и frame, а `executeAsyncScript()` ждёт callback. WebElement и другие arguments лучше передавать через `arguments`, а результат можно вернуть в Java. JavaScript полезен для специальных browser APIs и диагностики, но JS-click не должен постоянно обходить реальные проблемы с видимостью или перекрытием элемента.

10. Alert

   **Ответ:** На JavaScript alert, confirm или prompt переключаются через `driver.switchTo().alert()`, обычно после Explicit Wait `alertIsPresent()`. Интерфейс Alert позволяет получить текст, выполнить `accept()` или `dismiss()`, а для prompt — `sendKeys()`. Пока alert открыт, обычные действия со страницей могут завершаться `UnhandledAlertException`.

11. IFrame

   **Ответ:** Элементы внутри iframe недоступны из основного document context, поэтому сначала выполняется `driver.switchTo().frame()` по index, name/id или WebElement. Для возврата используются `parentFrame()` или `defaultContent()`. Frame нужно ожидать и выбирать явно; одинаковый locator в неверном context обычно приводит к `NoSuchElementException`.

12. WindowHandle

   **Ответ:** Window handle — уникальный идентификатор вкладки или окна в пределах WebDriver session. `getWindowHandle()` возвращает текущий handle, `getWindowHandles()` — все доступные, а `switchTo().window(handle)` меняет context. После закрытия текущего окна нужно переключиться на оставшийся handle, иначе следующая команда может вызвать `NoSuchWindowException`.

13. Selenium Exceptions

   **Ответ:** Часто встречаются `NoSuchElementException`, `StaleElementReferenceException`, `TimeoutException`, `ElementClickInterceptedException`, `ElementNotInteractableException`, `NoSuchFrameException` и `NoSuchWindowException`. Их нужно исправлять по причине: проверить locator и context, использовать подходящий Explicit Wait, заново найти stale element или убрать перекрытие. Универсальный `try/catch` и повтор всех действий скрывает дефекты и делает тесты flaky.

14. Методы классов Builder, Capabilities, Session

   **Ответ:** Builder собирает driver из адреса remote server и одного или нескольких вариантов Options, скрывая детали создания session. Capabilities позволяют задавать и читать параметры через browser Options, `setCapability()`, `getCapability()` и `asMap()`, а `merge()` объединяет наборы с осторожностью к конфликтам. Session имеет уникальный `SessionId`, который у `RemoteWebDriver` можно получить через `getSessionId()` и использовать для связи тестовых логов с Grid или cloud artifacts.

15. рассказать про любой фреймворк-обертку для селениума (Selenide, Frameworkium etc)

   **Ответ:** Selenide — Java wrapper над Selenium WebDriver с лаконичным API, встроенными smart waits, conditions, управлением browser lifecycle и автоматическими screenshots при ошибках. Например, проверка элемента формулируется через ожидаемое состояние, а Selenide ждёт его до timeout. Wrapper уменьшает boilerplate, но не отменяет понимание DOM, locators, WebDriver contexts и причин нестабильности.

16. Selenium Proxy для перехвата HTTP запросов и ответов

   **Ответ:** Proxy capability направляет browser traffic через заданный proxy, а специализированный proxy вроде BrowserMob Proxy может записывать и изменять HTTP traffic. В Selenium 4 network events и interception также развиваются через WebDriver BiDi; для Chromium доступны CDP-based возможности, но они привязаны к поддержке браузера и версии. Инструмент выбирают по задаче, а HTTPS interception может требовать установки доверенного сертификата.

17. Дублеры, Моки, Стабы

   **Ответ:** Test double — общее название объекта, заменяющего реальную dependency. Stub возвращает заранее заданные ответы и помогает управлять сценарием, mock дополнительно проверяет ожидаемые взаимодействия, а dummy только заполняет обязательный параметр и не участвует в логике. В UI test чаще подменяют внешний API через mock server или network interception, но сам браузер обычно оставляют реальным, иначе теряется цель проверки UI integration.

18. Selenium Grid

   **Ответ:** Selenium Grid принимает команды RemoteWebDriver и направляет каждую session на подходящий node по capabilities. Он позволяет параллельно запускать тесты на разных браузерах, версиях и операционных системах. Grid ускоряет набор, но требует контролировать capacity, очередь sessions, соответствие drivers, изоляцию данных, логи и очистку окружения.

19. Selenoid

   **Ответ:** Selenoid — реализация Selenium protocol от Aerokube, которая обычно запускает browser sessions в отдельных Docker containers. Браузеры и их версии описываются в `browsers.json`, а тест подключается через RemoteWebDriver endpoint; доступны VNC, video и logs при соответствующей настройке. Контейнеры дают изоляцию и быстрый parallel запуск, но инфраструктуре нужны Docker resources, заранее подготовленные images и ограничение числа sessions.

20. наиболее частые ошибки в UI/API тестах, как их решали

   **Ответ:** В UI tests частые проблемы — нестабильные locators, неправильные waits, общий driver или данные, неверный frame/window context и зависимость от порядка. В API tests — неправильный URL или environment, Authentication, headers, сериализация, несоответствие contract и некорректная подготовка данных. Я сначала локализую слой ошибки по request/response, browser console, screenshot и logs, воспроизвожу минимальный сценарий, затем исправляю причину; конкретный пример на собеседовании нужно заменить на `[ваша реальная ошибка и решение]`.

21. флаки тесты, как фиксили

   **Ответ:** Flaky test даёт разные результаты без изменения проверяемого кода. Я собираю artifacts и частоту падения, проверяю synchronization, данные, состояние окружения, порядок тестов, parallel execution и внешние зависимости, затем устраняю конкретную причину. Повторный запуск допустим как временная диагностическая мера, но постоянные retries не являются исправлением; реальный пример следует дать по шаблону `[причина — изменение — подтверждённый результат]`.
