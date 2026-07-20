# Mobile Automation

## Содержание

- [[#Вопросы и ответы]]

**Связанные заметки:** [[00 Индекс Skills Matrix AQA]]

---

## Вопросы и ответы

1. Android:

   **Ответ:** Android automation может выполняться на emulator или real device через native frameworks вроде UI Automator/Espresso либо через Appium driver UiAutomator2. Для подготовки нужны Android SDK, ADB, application package, device configuration и test runner. Выбор уровня зависит от риска: business logic лучше проверять ниже UI, а критичные flows — на устройствах.

2. Какие виды мобильных приложений бывают?(Android/iOS)

   **Ответ:** Native app создаётся для конкретной платформы и использует её UI/API, mobile web app работает в browser, hybrid app объединяет native shell и WebView. PWA является web application с возможностями установки и offline behavior, но не становится native app. Тип определяет automation context, доступные locators и способ сборки.

3. Что такое Appium?(Android/iOS)

   **Ответ:** Appium — open-source automation ecosystem и WebDriver server для управления разными platforms через устанавливаемые drivers. Для Android обычно используется UiAutomator2 driver, для iOS — XCUITest driver, который взаимодействует с WebDriverAgent. Test code отправляет WebDriver commands, а Appium переводит их в platform-specific automation.

4. Локаторы в Appium (Android/iOS)

   **Ответ:** Общие strategies включают accessibility id, id, class name и XPath. Для Android доступны UiAutomator expressions, для iOS — predicate strings и class chains в зависимости от driver. Лучше выбирать стабильный accessibility identifier или resource id; длинный XPath по hierarchy медленнее и ломается при изменении layout.

5. В чем отличие id от Accessibility ID (Android/iOS)

   **Ответ:** `id` обычно обращается к Android resource-id или соответствующему platform identifier, связанному с implementation. Accessibility ID ищет значение, предназначенное accessibility tree: Android `content-desc`, а в iOS — accessibility identifier. Accessibility ID удобен cross-platform при согласованных значениях, но identifier не должен ухудшать реальные accessibility labels для пользователя.

6. Activity/Package(Android)

   **Ответ:** Package идентифицирует Android application, например `com.example.app`, а Activity представляет конкретный экран или entry component. В Appium capabilities `appPackage` и `appActivity` могут запускать уже установленное приложение. Точные значения получают из manifest, ADB или информации текущей foreground activity.

7. Скоуп программ для запуска тестов(Android/iOS)

   **Ответ:** Для Android обычно нужны JDK, Android SDK/platform tools, ADB, emulator или device, Node.js, Appium server, UiAutomator2 driver, client library и build/test runner. Для iOS дополнительно нужны macOS, Xcode, Simulator или подписанное real device окружение и XCUITest driver. Appium Inspector помогает изучать hierarchy, но не заменяет server и test runner.

8. iOS:

   **Ответ:** Native iOS automation строится на XCTest/XCUITest, а Appium XCUITest driver управляет WebDriverAgent поверх этих технологий. Для запуска нужны Xcode, simulator или real device, signing и provisioning при необходимости. Ограничения Apple platform и version compatibility нужно проверять до проектирования CI.

9. XCUIApplication (start vs launch)

   **Ответ:** Создание `XCUIApplication` только описывает target application, а `launch()` запускает её как часть test. Стандартного метода `start()` у XCUIApplication нет; для уже запущенного приложения используется `activate()`, а для остановки — `terminate()`. Перед `launch()` задаются arguments и environment.

10. launchEnvironment, launchArguments

   **Ответ:** `launchArguments` передаёт application строки command line, а `launchEnvironment` — key-value environment variables. Их задают до `XCUIApplication.launch()` и используют для test mode, mock endpoints, locale или feature flags. Production behavior не должно случайно зависеть от небезопасного test-only flag.

11. XCUIElement

   **Ответ:** XCUIElement представляет UI element из accessibility hierarchy и позволяет tap, type, swipe и читать properties. Ссылка является proxy: наличие проверяется через `exists` или ожидание, а не гарантируется при создании query. Надёжность зависит от accessibility identifiers и корректной synchronization.

12. XCUIElementQuery

   **Ответ:** XCUIElementQuery лениво описывает поиск элементов по type, identifier, predicate и relationships. Query можно уточнять через matching и descendants, затем получить конкретный element или проверить count. Слишком широкий query и index-based выбор делают test хрупким.

13. XCTWaiter

   **Ответ:** XCTWaiter ожидает одну или несколько XCTestExpectation до timeout и возвращает детальный result, например completed или timedOut. Он полезен, когда нужно управлять порядком или не завершать test автоматически так, как это делает обычный wait API. UI element часто проще ждать через `waitForExistence(timeout:)`.

14. XCTestExpectations

   **Ответ:** XCTestExpectation представляет асинхронное событие, которое test должен дождаться, например callback или notification. Test создаёт expectation, код вызывает `fulfill()`, затем framework ждёт её с ограниченным timeout. Over-fulfill, неправильный count и отсутствие timeout превращают асинхронный test в нестабильный.

15. XCTAssert

   **Ответ:** XCTest предоставляет assertions вроде `XCTAssertTrue`, `XCTAssertEqual`, `XCTAssertNil` и `XCTFail`. Assertion должен сравнивать наблюдаемый результат с ожидаемым и иметь полезное сообщение. Несколько независимых причин падения лучше не смешивать в одном длинном scenario.

16. setUp(), tearDown() , AddTearDownBlock()

   **Ответ:** `setUp()` подготавливает состояние перед каждым test method, `tearDown()` очищает его после выполнения. `addTeardownBlock` регистрирует дополнительную cleanup operation во время test и удобно закрывает resource рядом с местом создания. Cleanup должен выполняться и после assertion failure, а общие mutable fixtures ухудшают parallelism.

17. accessibility labels, identifiers

   **Ответ:** Accessibility label — текстовое описание, которое слышит пользователь assistive technology, поэтому оно может локализоваться. Accessibility identifier предназначен для программной идентификации элемента и обычно не показывается пользователю. Для automation лучше стабильный identifier, а label тестируется отдельно как часть accessibility.

18. XCUIDevice

   **Ответ:** XCUIDevice представляет устройство и поддерживает операции вроде изменения orientation и нажатия hardware buttons, доступных automation API. Его используют для device-level interaction, а не поиска elements приложения. Некоторые системные возможности различаются между Simulator и real device.

19. Структура проекта и запуск (Appium):

   **Ответ:** Проект обычно разделяет tests, Screen Objects, components, driver factory, configuration, data и reporting. Appium server и platform driver запускаются отдельно или управляются infrastructure code, а test runner создаёт session по capabilities. Локальный и remote запуск используют один test API, меняя endpoint и configuration.

20. структура Appium-проекта (Page Object или Screen Object)

   **Ответ:** Для mobile чаще говорят Screen Object: class описывает экран через locators, actions и состояние. Повторяемые widgets выносятся в Component Objects, а platform-specific locators можно скрыть за общим interface только при одинаковом поведении. Test должен выражать business flow, а не напрямую вызывать driver в каждой строке.

21. как запускать Appium локально (через UI и CLI)

   **Ответ:** Актуальный Appium server устанавливается и запускается CLI-командой `appium`, а drivers отдельно устанавливаются через `appium driver install`. UI-инструмент Appium Inspector подключается к уже доступному server и создаёт session для исследования. Старые Appium Desktop distributions могут не соответствовать современным server/drivers, поэтому версии нужно проверять.

22. запуск тестов через Appium Inspector

   **Ответ:** Appium Inspector не является полноценным test runner: он создаёт Appium session, показывает source hierarchy, позволяет выполнять отдельные actions и подбирать locators. Из него можно получить code snippets, после чего test запускается через JUnit, TestNG, pytest или другой runner. Inspector полезен для диагностики, но не подтверждает стабильность полного scenario.

23. конфигурация appium.conf.js или .yaml

   **Ответ:** Appium server поддерживает JSON, YAML, JS и CJS configuration; автоматически обнаруживаются имена вроде `.appiumrc.yaml` и `appium.config.js`. В файле задаются server options и driver/plugin configuration, а CLI values имеют приоритет. Session capabilities обычно остаются в test configuration или provider setup, а secrets не помещаются в общий файл.

24. Android:

   **Ответ:** Для Android Appium session выбирает installed driver, обычно UiAutomator2, device по UDID/deviceName и application через app path либо package/activity. ADB обеспечивает связь с device, установку и сбор logs. В parallel run каждому device нужны уникальные server-side ports и независимые данные.

25. Обязательные capability(Android/iOS)

   **Ответ:** Минимальный набор зависит от Appium driver и provider, но обычно включает W3C `platformName` и `appium:automationName`, а также способ выбрать device и приложение. Для Android это могут быть `appium:deviceName`/`udid`, `appium:app` или package/activity; для iOS — device, platformVersion и app/bundleId. Non-standard capabilities используют prefix `appium:` или vendor namespace.

26. MobileElement

   **Ответ:** `MobileElement` использовался в старых версиях Appium Java Client, но удалён начиная с major version 8. В актуальном code обычно применяется Selenium `WebElement`, а mobile-specific commands находятся у AndroidDriver/IOSDriver или extension interfaces. Возвращать старую dependency только ради MobileElement не стоит.

27. AppiumDriver (Android/iOS)

   **Ответ:** AppiumDriver — базовый Java client driver для Appium sessions, а AndroidDriver и IOSDriver дают platform-specific возможности. Driver получает remote server URL и Options/capabilities, отправляет WebDriver commands и завершает session через `quit()`. Один instance нельзя безопасно делить между parallel threads.

28. UiAutomator2 (Android)

   **Ответ:** UiAutomator2 driver — официальный Appium driver для Android, который переводит WebDriver commands в Android automation через UiAutomator2 server. Он поддерживает native и web/hybrid contexts, device commands и Android-specific locator strategies. Driver устанавливается как Appium extension и имеет собственные version requirements и capabilities.

29. iOS:

   **Ответ:** Для Appium на iOS используется XCUITest driver и WebDriverAgent, который должен быть собран и подписан для target device. Simulator проще для CI, а real device нужен для hardware, push, camera и части system behavior. Signing, ports и derived data становятся отдельными рисками parallel execution.

30. XCUIApplication (start vs launch)

   **Ответ:** `launch()` запускает XCUIApplication с настроенными arguments/environment и обычно сбрасывает её process для test. Метода `start()` в публичном XCUIApplication API нет; `activate()` переводит установленное приложение в foreground без нового launch flow. Выбор зависит от того, проверяется cold start или возврат к существующей session.

31. launchEnvironment, launchArguments

   **Ответ:** Эти свойства задаются до `launch()` и передают application test configuration. Arguments удобны для flags, environment — для именованных values, но sensitive data не следует выводить в logs. App должен явно поддерживать test mode и не включать его в production случайно.

32. XCUIElement

   **Ответ:** XCUIElement предоставляет actions и properties элемента accessibility tree. Перед interaction я жду `exists` и нужное состояние, а после изменения UI при необходимости заново получаю element из query. Identifier-based поиск стабильнее coordinates.

33. XCUIElementQuery

   **Ответ:** Query фильтрует элементы по type, identifier, predicate и hierarchy и вычисляется при обращении к result. Его можно переиспользовать как описание поиска, но не следует полагаться на случайный `element(boundBy:)`, если порядок UI меняется. Уникальность лучше подтверждать в test design.

34. XCTWaiter

   **Ответ:** XCTWaiter управляет ожиданием expectations и позволяет анализировать результат без немедленного завершения test. Он поддерживает timeout и порядок, если это требуется scenario. Для обычного UI появления предпочтителен более прямой element wait.

35. XCTestExpectations

   **Ответ:** Expectations синхронизируют asynchronous code с test, например callback, notification или predicate. Каждая expectation должна fulfill-иться ожидаемое число раз до общего timeout. После ожидания test проверяет сам результат, потому что факт callback не всегда означает правильные данные.

36. XCTAssert

   **Ответ:** Assertions фиксируют expected behavior и останавливают или помечают test при расхождении согласно framework behavior. Я выбираю наиболее конкретный XCTAssert и добавляю context. Проверку нельзя заменять только screenshot или log.

37. setUp(), tearDown() , AddTearDownBlock()

   **Ответ:** Setup создаёт application и данные, teardown закрывает session и очищает state. `addTeardownBlock` регистрирует cleanup для resource, созданного внутри test или setup. Блоки не должны скрывать исходное падение новой ошибкой очистки.

38. accessibility labels, identifiers

   **Ответ:** Label описывает элемент пользователю VoiceOver и может зависеть от языка, identifier стабильно связывает automation с element. Одинаковый identifier не должен без необходимости повторяться на одном экране. Наличие identifier не заменяет проверку accessibility behavior.

39. XCUIDevice

   **Ответ:** Через XCUIDevice можно менять orientation и выполнять поддерживаемые device button actions. Это помогает проверять rotation и system interaction, но API не даёт полного контроля над реальным hardware. Возможность следует проверять для конкретных Xcode и device types.

40. Interruption monitor

   **Ответ:** XCTest UI interruption monitor регистрирует handler для системных alerts, например permission dialog. После регистрации test выполняет interaction, чтобы XCTest обнаружил interruption, а handler находит и нажимает нужную кнопку. Monitor удаляется после использования и не должен автоматически принимать любой alert без проверки текста.

41. XCTContext.runActivity

   **Ответ:** `XCTContext.runActivity` группирует действия в именованную activity внутри XCTest report. К activity можно прикладывать screenshot, text и другие XCTAttachment, что упрощает диагностику. Activity структурирует отчёт, но не заменяет assertions.

42. Интеракции и нестандартные сценарии:

   **Ответ:** К ним относятся system permissions, notifications, background/foreground, orientation, locale, deep links, biometrics и network changes. Возможности различаются между Appium driver, native XCTest, emulator/simulator и real device. Scenario нужно сначала проверить на поддерживаемость выбранной infrastructure.

43. работа с системными алертами (permissions, push notifications) через Appium и InterruptionMonitor

   **Ответ:** В Appium alert можно обработать через native context, platform locators и соответствующие mobile commands или заранее настроить permissions; automatic accept применяют осторожно. `InterruptionMonitor` относится к native XCUITest code и обрабатывает iOS interruptions по проверяемому тексту и кнопке. Push notification testing дополнительно требует signing, entitlement, server payload и часто real device.

44. тестирование в background/foreground режимах (runAppInBackground)

   **Ответ:** Appium clients предоставляют command для временного background, а также activate/terminate/query app state в зависимости от driver. Test фиксирует состояние до ухода, ожидает нужное событие и проверяет восстановление после foreground. Точное имя метода и parameter type зависят от версии client, поэтому wrapper должен изолировать API change.

45. изменение ориентации экрана, смена языков и регионов

   **Ответ:** Orientation можно менять driver command или XCUIDevice, а locale/language задавать capabilities, simulator settings или перезапуском device согласно platform. После изменения проверяются layout, resources, formats и сохранение state. Не все cloud devices разрешают runtime change, поэтому matrix иногда создаёт отдельную session на каждую configuration.

46. Опыт настройки CI для мобильной автоматизации(Android/iOS):

   **Ответ:** Нужен реальный шаблон: «Pipeline собирал app, запускал emulator/simulator или подключался к farm, выполнял tests и публиковал logs/video/screenshots». Для iOS agent должен быть macOS с совместимым Xcode и signing, для Android — SDK и managed AVD/device. Cleanup и retry infrastructure failures отделяются от product failures.

47. параметризация запусков

   **Ответ:** Pipeline передаёт platform, device, OS version, app artifact, locale, orientation, tags и remote endpoint через typed configuration. Matrix формируется по risk и usage analytics, а не как декартово произведение всех values. Session metadata должна попадать в report.

48. отчёты (Allure/TestNG/JUnit + скриншоты, логи, видео)

   **Ответ:** TestNG/JUnit предоставляет execution results, Allure добавляет steps и attachments, а device infrastructure — video, logcat/syslog и device logs. Все artifacts связываются с test, device, session id и build. Tokens, user data и notification contents маскируются.

49. Appium Grid / Device Farm запуск

   **Ответ:** Test создаёт Remote Appium session на Grid или provider endpoint с platform и vendor capabilities. Infrastructure выбирает node/device, устанавливает app и возвращает session id, по которому доступны artifacts. Нужно учитывать parallel quota, очередь, уникальные ports on-premises и provider-specific features.

50. Работа c фермой(Android/iOS):

   **Ответ:** Обычно app загружается через UI/API, затем выбираются devices/OS и test framework либо Appium endpoint. Run отслеживается по build/session metadata, после чего скачиваются logs, video, screenshots и results. Ограничения real devices, signing, network tunnel и data cleanup проверяются заранее.

51. параметризация, отчёты, запуск на ферме

   **Ответ:** Device matrix и test shards передаются из CI, credentials берутся из secret storage, app загружается один раз и получает versioned identifier. Framework создаёт sessions, обновляет их status и после run получает artifacts через API. Report агрегирует результаты, сохраняя platform-specific failures и queue time.

52. Паралелльный запуск автоматизированных мобильных тестов (Android/iOS)

   **Ответ:** Каждый parallel worker получает отдельный device, driver, test account и data namespace. На локальном Appium для Android разделяются `systemPort`, а для iOS — WebDriverAgent/MJPEG ports и derived data по требованиям driver; на farm действует purchased concurrency. Tests не должны зависеть от порядка, общей clipboard/session или одного backend record.
