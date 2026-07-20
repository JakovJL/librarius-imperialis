# Mobile Device Farms

## Содержание

- [[#Вопросы и ответы]]

**Связанные заметки:** [[00 Индекс Skills Matrix AQA]]

---

## Вопросы и ответы

1. Эмуляторы и симуляторы:

   **Ответ:** Виртуальные устройства быстро создаются, удобны для development и дают управляемую configuration. Real devices нужны для hardware, OEM modifications, производительности, sensors, camera, push и реального network behavior. Эффективная стратегия использует много быстрых virtual runs и выбранную матрицу физических устройств.

2. отличия эмуляторов и симуляторов

   **Ответ:** Emulator стремится воспроизвести hardware и OS environment, а simulator имитирует platform behavior без полного воспроизведения hardware. Android Emulator запускает Android virtual device, iOS Simulator выполняет simulator build и не является настоящим iPhone. Оба отличаются от real device по architecture, drivers, performance и доступным system capabilities.

3. как запускать тесты на эмуляторе (Android Studio, AVD)

   **Ответ:** В Android Studio Device Manager создаётся AVD с device profile, system image и API level, затем он запускается и появляется в `adb devices`. Test устанавливает APK или Appium делает это по capabilities и выбирает emulator по UDID/deviceName. В CI лучше использовать versioned image, hardware acceleration, readiness check и snapshot только при контролируемом состоянии.

4. как подключать к фермам (через CLI или UI)

   **Ответ:** Через UI обычно загружают app/test package, выбирают device matrix и запускают run. Через CLI или REST API pipeline загружает artifacts, получает их id, создаёт run/sessions и опрашивает status. Для Appium можно подключиться к remote endpoint с credentials и vendor capabilities.

5. Целесообразность применения device farms

   **Ответ:** Farm полезна, когда нужна широкая матрица реальных devices/OS, parallel execution и доступ без поддержки собственного lab. Она особенно ценна для fragmentation Android, нескольких iOS versions и воспроизведения device-specific defects. Для частых tests на двух известных моделях local devices могут быть быстрее и дешевле.

6. Преимущества и недостатки применения

   **Ответ:** Преимущества — большой catalog, real hardware, parallelism, remote access, video/logs и отсутствие физического обслуживания. Недостатки — стоимость, queue, network latency, ограничения permissions, provider outages, security требования и сложнее live debugging. Test strategy должна учитывать quota и не запускать полную матрицу на каждый commit.

7. Перечислить известные фермы:

   **Ответ:** Известны BrowserStack, Sauce Labs, Firebase Test Lab, AWS Device Farm, SmartBear BitBar, Kobiton и LambdaTest. У них различаются catalogs, physical/virtual devices, frameworks, pricing, private devices, network tunnels и artifacts. Перед выбором нужно проверить актуальную поддержку конкретной platform и scenario.

8. BrowserStack App Live / App Automate

   **Ответ:** App Live предназначен для интерактивного manual testing приложения на remote device, а App Automate — для automated tests, включая Appium, XCUITest и Espresso. App Automate группирует sessions по project/build и предоставляет video, device/network logs и REST API. Конкретные capabilities и concurrency зависят от плана и актуальной документации.

9. Sauce Labs

   **Ответ:** Sauce Labs предоставляет real и virtual device cloud для manual и automated mobile testing с Appium и native frameworks. Интеграция использует remote endpoint, app storage, capabilities и CI metadata. Перед внедрением я бы проверил доступные devices, tunnels, artifacts, concurrency и data residency.

10. Firebase Test Lab

   **Ответ:** Firebase Test Lab запускает Android и iOS tests на устройствах Google lab; Android также поддерживает virtual devices. Tests можно запускать из Firebase console, Android Studio, `gcloud` и CI, задавая matrix model/OS/locale/orientation. Сервис поддерживает native test types, а не является универсальным Appium endpoint для любого scenario.

11. AWS Device Farm

   **Ответ:** AWS Device Farm предоставляет реальные Android, iOS и Fire OS devices, automated runs и remote access. Можно загрузить app и test package либо использовать managed Appium endpoint для remote session согласно доступному режиму. Нужно учитывать IAM, supported frameworks, artifacts, pricing и региональные ограничения сервиса.

12. BitBar (от SmartBear)

   **Ответ:** BitBar — SmartBear device/browser cloud с real devices, manual и automated testing и интеграцией через API/CI. Он поддерживает Appium и native frameworks, а также public, dedicated и private deployment варианты. Выбор проверяется proof of concept на нужном app signing, network и artifacts.

13. Kobiton

   **Ответ:** Kobiton предоставляет real device access для manual и Appium automation, включая cloud и enterprise deployment варианты. Платформа ориентирована на session artifacts и управление device lab. Перед использованием нужно подтвердить поддержку нужных OS versions, private network, concurrency и API.

14. LambdaTest (c Appium)

   **Ответ:** LambdaTest предоставляет remote real-device mobile testing и запуск Appium sessions через vendor endpoint и capabilities. App загружается в storage provider, а CI получает app id и запускает device matrix. Vendor-specific options лучше изолировать в configuration adapter.

15. Работа с логами и отчетами:

   **Ответ:** Для каждого run сохраняются test runner report, session video, screenshots, device logs, Appium/server logs и при возможности network data. Artifacts связываются по session id, test name, build и device. Сначала анализируется timeline, затем определяется product, automation или infrastructure failure.

16. уметь получить логи и скриншоты из фермы

   **Ответ:** Их можно открыть в dashboard конкретной session или скачать через provider API после завершения run. Pipeline должен получать artifacts даже при failed test и сохранять ссылки в общем report. Retention и permissions проверяются заранее, а sensitive data маскируются.

17. понимать, где искать crash logs или ошибки инициализации

   **Ответ:** Crash ищут в device/system logs, crash report и приложенном stack trace, а session initialization — в Appium/provider logs и capability validation error. Также проверяются upload app, signing, device availability, OS compatibility, installation и startup activity/bundle. Error до создания session обычно не появится в test step report.

18. Конфигурация устройств:

   **Ответ:** Device configuration включает model, OS, form factor, locale, orientation, region, network, permissions и app state. Не все provider features доступны на всех devices. Matrix должна быть versioned и попадать в report для воспроизводимости.

19. выбор конкретных моделей

   **Ответ:** Модели выбирают по analytics пользователей, business-critical сегментам, screen sizes, chipset/OEM differences и известным рискам. Обычно есть небольшая Smoke matrix и более широкая nightly/release matrix. Редкое устройство добавляется при high-impact defect, а не только ради количества.

20. ОС, язык, ориентация, регион, геолокация

   **Ответ:** Эти dimensions задаются capabilities или configuration provider, если конкретное device поддерживает изменение. Tests проверяют locale resources, date/number formats, layout rotation, regional content и location permissions. Каждая комбинация повышает стоимость, поэтому используется pairwise/risk-based selection, а не полный декартов продукт.

21. Логгирование:

   **Ответ:** Нужны logs test framework, Appium/client, provider, device OS и application с общей временной шкалой. Correlation id связывает UI action с backend request. Уровень logs настраивается так, чтобы диагностировать failure без утечки token, PII и лишнего объёма.

22. сбор и анализ Logcat (Android)

   **Ответ:** Logcat содержит system и application messages с time, process, tag и level. Я фильтрую по package/PID, crash markers и времени test step, сохраняя raw fragment как artifact. На shared device buffer может содержать посторонние данные, поэтому scope и очистка требуют осторожности.

23. syslog/XCTest logs (iOS)

   **Ответ:** iOS device/simulator logs показывают system и application events, а XCTest result bundle содержит test activities, failures и attachments. Я сопоставляю timestamps, process и test case, а для Appium добавляю WebDriverAgent log. Доступность syslog зависит от provider и device policy.

24. анализ crash logs

   **Ответ:** Сначала определяю exception/signal, affected process, OS/device и момент crash, затем symbolicate stack trace подходящими dSYM/build symbols. Crash без правильных symbols мало полезен. Также проверяются memory pressure, watchdog termination и breadcrumbs до падения.

25. логирование из тестов

   **Ответ:** Test logs фиксируют business step, sanitized parameters, start/stop и result, а не каждую внутреннюю строку framework. Session id и device metadata добавляются автоматически. Ошибка должна прикладывать последние actions и artifacts, сохраняя исходный stack trace.

26. Дебаг тестов:

   **Ответ:** Debug начинается с воспроизводимости: app build, device/OS, capabilities, test data и provider session. Затем сравниваются video, steps, device/Appium logs и backend telemetry. Временный retry помогает определить flakiness, но не считается исправлением.

27. анализ падений

   **Ответ:** Failure классифицируется как product defect, test defect, data problem, environment/provider issue или unsupported capability. Я нахожу первое отклонение на timeline, а не последнюю exception, и проверяю похожие sessions. После исправления повторяю test на том же device и на контрольной configuration.

28. воспроизведение через Live-сессию

   **Ответ:** Live session позволяет вручную открыть то же device/OS, установить тот же app build и повторить steps. Она полезна для визуальных, permission и device-specific проблем. Manual успех не опровергает race condition, поэтому logs и automated timing всё равно анализируются.

29. использование remote debugger

   **Ответ:** Remote debugger подключается к поддерживаемому WebView/browser или application process через provider feature и platform tools. Он помогает смотреть DOM, console, network и breakpoints, но может менять timing и не всегда доступен на real device. Доступ ограничивается, а sensitive production data не используются.

30. Запись видео

   **Ответ:** Video показывает фактический UI, orientation, системные dialogs и момент зависания. Оно связывается с test timeline и полезно вместе с logs, но само не объясняет backend или synchronization cause. Recording может влиять на performance и содержать PII, поэтому имеет retention policy.

31. Снятие скриншотов

   **Ответ:** Screenshot снимается при failure и при ключевых состояниях, а не после каждого действия без необходимости. В имени или metadata сохраняются test, device, step и timestamp. Для visual comparison дополнительно фиксируются resolution, orientation, locale и допустимые differences.

32. Параллельный запуск тестов:

   **Ответ:** Parallel execution создаёт независимую session на worker/device и требует изолированных accounts, data и artifacts. Provider ограничивает concurrency plan и может ставить excess sessions в queue. Framework должен безопасно собирать результаты независимо от порядка завершения.

33. умение распараллеливать прогон на десятках/сотнях девайсов

   **Ответ:** Suite делится на balanced shards, workers получают device matrix из scheduler, а concurrency ограничивается quota и backend capacity. App upload и shared setup выполняются один раз, sessions создаются с idempotent retry только при infrastructure error. Масштабирование нужно нагрузочно проверить, чтобы tests не перегрузили общий API.

34. device matrix

   **Ответ:** Device matrix — набор model, OS, locale, orientation и других dimensions, на которых выполняется test scope. Она строится по usage analytics, supported versions, risk и defect history. Matrix пересматривается при изменении аудитории и catalog provider.

35. шардирование

   **Ответ:** Sharding делит suite на независимые части для parallel workers. Балансировать лучше по исторической duration, сохраняя связанные setup только там, где это необходимо. Каждый shard публикует отдельный result, затем aggregator формирует общий status и обнаруживает потерянный shard.

36. Настройка окружения для удаленного запуска тестов:

   **Ответ:** Нужны provider account/project, app artifact, remote endpoint, credentials, network tunnel и capabilities. CI runner должен видеть provider и при необходимости private backend, а test device — test environment. Versions client/Appium/provider фиксируются и выводятся в report.

37. подключение cloud-фермы к пайплайну (Jenkins, GitHub Actions, GitLab)

   **Ответ:** Pipeline собирает app, загружает её API/CLI, сохраняет app id, формирует matrix и запускает tests с remote endpoint. Затем ждёт sessions, собирает status/artifacts и удаляет temporary tunnel. Job status отражает tests, а provider outage классифицируется отдельно.

38. использование токенов/секретов для безопасной авторизации

   **Ответ:** Username/access key или token хранятся в CI secret store, выдаются только нужной job и маскируются в logs. Предпочтительны scoped и ротируемые credentials, если provider их поддерживает. Secrets не передаются в repository, screenshots, command arguments с публичным выводом или report.

39. автоматическая загрузка артефактов (видео, логи, скриншоты) в CI

   **Ответ:** После session framework или отдельный step вызывает provider API по session id и скачивает доступные artifacts. Шаг выполняется в `always/finally`, проверяет checksum/size и сохраняет manifest со ссылками. Retention в CI и provider согласуется, чтобы ссылки не стали пустыми раньше отчёта.

40. использование CLI-инструментов (например, browserstack-cli, bs-local)

   **Ответ:** CLI или provider binary автоматизирует app upload, Local tunnel, запуск и получение results. Точное название и flags меняются, поэтому version фиксируется и проверяется через официальный help. Tunnel запускается до tests, проходит health check и гарантированно останавливается после job.

41. параллельный запуск и управление сессиями

   **Ответ:** Session manager ограничивает concurrency, хранит mapping test → session id и завершает orphaned sessions. Queue имеет timeout, а retry создания session не дублирует уже запущенный test. Status provider обновляется passed/failed только после корректного test result.

42. Сбор метрик по результатам тестовых прогонов:

   **Ответ:** Полезны pass/fail/inconclusive, duration, queue time, device allocation time, flaky rate, infrastructure failure rate и стоимость. Метрики группируются по device, OS, app build, suite и provider. Они помогают менять matrix и capacity, но не должны скрывать critical individual defect средним процентом.

43. сбор артефактов (видео, скриншоты, логи)

   **Ответ:** Artifacts собираются автоматически для failed и выбранных successful sessions и имеют единый metadata manifest. Доступ проверяется после загрузки, а format сохраняет raw evidence. Sensitive captures очищаются или защищаются согласно policy.

44. агрегация данных

   **Ответ:** Aggregator объединяет runner results и provider session data по stable test id, build и device configuration. Duplicate retries не должны считаться отдельными product tests без отметки. Общая сводка позволяет drill-down до конкретного video и log.

45. отчёты по стабильности

   **Ответ:** Отчёт показывает history результата одного test на одинаковых и разных devices, flaky/infrastructure classifications и duration trend. Стабильность provider/device отделяется от стабильности test и продукта. Решения принимаются по достаточно длинному периоду, а не одному run.

46. flaky rate

   **Ответ:** Flaky rate можно считать как долю tests, изменивших результат при retry без изменения app build, но формула и denominator должны быть зафиксированы. Метрика разбивается по test, device, OS и cause. Retry не превращает flaky test в passed без отдельной отметки.

47. Масштабирование:

   **Ответ:** Масштабируются workers, device concurrency, artifact processing и backend test data. Bottleneck измеряется в queue, setup, execution и upload stages. Больше devices не ускорят suite с serial dependency или одним shared account.

48. динамическое распределение нагрузок

   **Ответ:** Scheduler выдаёт следующий shard доступному compatible device с учётом capability, duration и provider capacity. Worker heartbeat и lease позволяют вернуть shard в queue при infrastructure failure. Business test не должен выполняться дважды одновременно без изоляции side effects.

49. управление сессиями через API

   **Ответ:** Provider API позволяет получать session status и artifacts, обновлять metadata/result и иногда завершать session. Client должен обрабатывать rate limits, pagination, transient errors и eventual availability artifacts. Все операции логируются с session id без публикации credentials.

50. Поддержка разных провайдеров и переключение между ними

   **Ответ:** Общий interface скрывает endpoint, upload API, vendor capabilities и artifact client, а core tests используют стандартные Appium options. Provider-specific features остаются в adapters и capability profiles. Полная прозрачность невозможна из-за разных catalogs и возможностей, поэтому переключение подтверждается contract/smoke tests и не сводится к замене URL.
