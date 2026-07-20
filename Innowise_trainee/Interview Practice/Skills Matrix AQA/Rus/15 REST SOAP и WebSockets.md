# REST, SOAP и WebSockets

## Содержание

- [[#Вопросы и ответы]]

**Связанные заметки:** [[00 Индекс Skills Matrix AQA]]

---

## Вопросы и ответы

1. Базовые понятия:

   **Ответ:** При тестировании API нужно понимать transport, protocol или architectural style, формат сообщений, contract и модель взаимодействия. REST обычно использует HTTP request/response, SOAP обменивается формализованными XML messages, а WebSocket поддерживает длительное двустороннее соединение. Для каждого варианта проверяются функциональность, ошибки, безопасность, производительность и совместимость.

2. что такое REST, SOAP, WebSocket

   **Ответ:** REST — architectural style, в котором resources имеют URI, а операции используют единый interface и stateless requests. SOAP — protocol обмена XML messages с формальным Envelope и расширениями вроде WS-Security. WebSocket — protocol поверх TCP с HTTP-compatible handshake, после которого client и server могут независимо отправлять text или binary messages по одному соединению.

3. основные отличия REST ↔ SOAP

   **Ответ:** REST не задаёт единый формат сообщения и чаще использует HTTP с JSON, тогда как SOAP строго использует XML message structure и может работать поверх разных transports. SOAP имеет стандарты WS-* для security, reliability и transactions; REST обычно проще и легче интегрируется с web ecosystem. Выбор зависит от contract, инфраструктуры и требований, а не от утверждения, что один подход всегда лучше.

4. идемпотентность и HTTP-методы (GET, POST, PUT, PATCH, DELETE)

   **Ответ:** Метод идемпотентен, если несколько одинаковых requests имеют тот же предполагаемый effect на server, что и один. GET является safe и idempotent, PUT и DELETE по HTTP semantics идемпотентны, а POST и PATCH не гарантируют идемпотентность. Повторный DELETE может вернуть другой status, но итоговое состояние resource остаётся тем же; для безопасного retry POST иногда используют idempotency key.

5. HTTP статус-коды и структура запроса/ответа

   **Ответ:** Request содержит method, target URI, HTTP version, headers и optional body; response — status code, headers и optional body. Классы codes: 1xx informational, 2xx success, 3xx redirection, 4xx client error и 5xx server error. В тесте я проверяю не только status, но и headers, content type, schema, body и side effects.

6. JSON vs. XML

   **Ответ:** JSON компактно представляет objects, arrays и primitive values и широко используется в web API. XML поддерживает attributes, namespaces, mixed content и строгие schemas вроде XSD, но обычно многословнее. Форматы нужно сравнивать по contract: преобразование XML в JSON может потерять различие между attribute, element и namespace.

7. разница GET/POST, PUT/POST

   **Ответ:** GET получает representation и по semantics не должен изменять server state, а POST отправляет данные для обработки и часто создаёт subordinate resource или запускает command. PUT создаёт или полностью заменяет resource по известному URI и должен быть идемпотентным. POST обычно не идемпотентен и часто позволяет server самому определить URI нового resource.

8. Инструменты для отправки запросов:

   **Ответ:** Для ручного исследования подходят Postman, curl, browser DevTools и специализированные SOAP или WebSocket clients. Для автоматизации используются language libraries и test frameworks. Инструмент должен позволять контролировать method, URL, headers, body, Authentication, certificates, timeouts и сохранять фактический response.

9. отправка HTTP-запросов через Postman, создание коллекций

   **Ответ:** В Postman создаётся request с method, URL, parameters, headers, body и Authentication, после чего проверки можно добавить в post-response scripts. Связанные requests объединяются в collection, а значения выносятся в environment или collection variables. Secrets не следует экспортировать вместе с collection, а запуск в CI выполняется через поддерживаемый CLI с публикацией отчёта.

10. работа с curl

   **Ответ:** curl отправляет request из command line, например method задаётся через `-X`, headers через `-H`, body через `--data`, а подробный обмен виден с `-v`. Для JSON обычно указывают `Content-Type: application/json`, а token передают в Authorization header. В логах CI нельзя печатать secrets, и shell quoting нужно учитывать отдельно для PowerShell и Unix-like shells.

11. чтение API-документации (Swagger/OpenAPI)

   **Ответ:** Я начинаю с servers и security schemes, затем читаю path, method, parameters, request body, response codes и schemas. Swagger UI — интерфейс для просмотра и отправки requests, а OpenAPI document — machine-readable описание API. Документация задаёт ожидаемый contract, но тесты также должны проверять реальные business rules, ошибки и права доступа.

12. просмотр запросов в DevTools (Chrome/Firefox)

   **Ответ:** Во вкладке Network можно увидеть URL, method, status, request и response headers, payload, timing, cookies и initiator. Фильтры Fetch/XHR и WS помогают отделить API calls и WebSocket frames. Sensitive headers и данные нужно маскировать перед сохранением HAR или screenshot.

13. Сериализация, десериализация:

   **Ответ:** Serialization преобразует object в transport format вроде JSON или XML, а deserialization создаёт object из полученных данных. Ошибки возникают из-за типов, дат, null, неизвестных fields, naming policy и несовместимой schema. В тестах полезно отдельно проверять raw response и typed model, чтобы строгая deserialization не скрыла фактический ответ server.

14. уметь обрабатывать JSON, XML и использовать библиотеки (например, Jackson, Gson, Newtonsoft, etc.)

   **Ответ:** В Java Jackson или Gson преобразуют JSON в objects и обратно, а Jackson также имеет XML module; в .NET часто используется Newtonsoft.Json или System.Text.Json. Нужно уметь настроить field mapping, dates, unknown properties и custom converters. Library version и настройки должны быть едиными с contract, а untrusted data нельзя десериализовать небезопасным polymorphic способом.

15. Использование библиотек для работы с протоколами:

   **Ответ:** Library выбирается по protocol, language и уровню abstraction. Важно контролировать connection pooling, timeouts, TLS, retries, serialization, logging и async behavior. Обёртка проекта должна добавлять domain-specific operations, но не скрывать status, headers и raw response, необходимые для диагностики.

16. RestAssured, Retrofit, Requests (Python), HttpClient

   **Ответ:** REST Assured — Java DSL, удобный для API assertions; Retrofit генерирует typed HTTP client по Java interfaces и converters. Python Requests предоставляет простой synchronous client, а название HttpClient используется в нескольких стеках, например Java и .NET, для низкоуровневой отправки HTTP. Ни один client не заменяет проверку contract и business assertions.

17. WebSoccets:

   **Ответ:** Правильное название технологии — WebSockets. При тестировании важны handshake, Authentication, subprotocol, text и binary messages, порядок событий, reconnect, ping/pong и корректное закрытие. Поскольку сообщения приходят асинхронно, тесту нужны thread-safe collection, filtering и ограниченные waits.

18. что такое и как работает

   **Ответ:** Client начинает с HTTP handshake и просит перейти на WebSocket protocol; после успешного ответа создаётся двусторонний канал. Данные передаются frames, которые образуют text, binary и control messages, а обе стороны могут отправлять их независимо. Соединение завершается closing handshake или обрывом, который client должен корректно обработать.

19. работа с WebSocket (через wscat/web-интерфейс)

   **Ответ:** wscat позволяет подключиться к `ws://` или защищённому `wss://` endpoint, передать headers и вручную отправлять и читать messages. Browser interface или DevTools показывает handshake и frames реального приложения. Перед проверкой нужно знать Authentication, subprotocol и application message format, потому что WebSocket сам их не определяет.

20. уметь подключаться, слушать, обрабатывать события, проверять сообщения.

   **Ответ:** Client регистрирует handlers для open, message, error и close, затем подключается и отправляет нужное сообщение или subscription. Полученные events складываются в thread-safe queue, где тест ждёт сообщение нужного type и проверяет schema и fields. В teardown connection закрывается, а listener освобождается даже после падения теста.

21. реализация клиентов WebSocket для автотестов (на Java/Python/JS)

   **Ответ:** В Java можно использовать стандартный `java.net.http.WebSocket` или library проекта, в Python — подходящий WebSocket client, в JavaScript — browser `WebSocket` или Node.js library. Client wrapper обычно отвечает за connect, authentication, send, queue сообщений, filtering, timeout и close. Выбор должен поддерживать TLS, headers и subprotocol, необходимые системе.

22. таймауты, ожидание и фильтрация нужных сообщений

   **Ответ:** Тест ждёт не первое сообщение, а event с нужным type, correlation id и business key до общего deadline. Неподходящие events сохраняются или пропускаются по явному правилу, а timeout report показывает все полученные сообщения без secrets. `Thread.sleep()` ненадёжен, потому что не синхронизирован с фактическим событием.

23. Работа с REST:

   **Ответ:** REST testing включает проверку resources, methods, status codes, representations, headers, Authentication, authorization и error contract. Отдельно проверяются idempotency, pagination, filtering, concurrency и cache semantics, если они заявлены. Tests должны быть независимыми и управлять своими данными через API или fixtures.

24. умение валидировать JSON-схемы ответа (JSON Schema, Ajv, fastjsonschema)

   **Ответ:** JSON Schema проверяет types, required properties, formats, ranges и структуру response. Ajv используется в JavaScript, fastjsonschema — в Python; в других языках есть собственные validators. Schema validation не проверяет правильность business values, поэтому после неё нужны semantic assertions, а версия dialect должна совпадать со schema.

25. поддержка разных методов аутентификации (Bearer-token, Basic Auth, OAuth2)

   **Ответ:** Basic Auth передаёт base64-encoded credentials в Authorization header и безопасен только поверх TLS. Bearer token также передаётся в header, но даёт доступ любому предъявителю, поэтому его нужно защищать. OAuth 2.0 описывает получение делегированного access token; test client должен использовать flow, разрешённый для его типа, и не хранить secrets в коде.

26. обработка ошибок и нестандартных ответов (например, пустой body + статус 204)

   **Ответ:** Test должен сначала интерпретировать status и headers, а затем разбирать body только когда он разрешён contract. Response `204 No Content` не должен содержать message body, поэтому попытка обязательной JSON deserialization является ошибкой client. Для error responses проверяются стабильный error code, message, field details и отсутствие sensitive information.

27. Безопасность и инфраструктура:

   **Ответ:** API требует TLS, безопасного Authentication, точной authorization, input validation, rate limiting, secret management и audit logs. Инфраструктура должна контролировать certificates, gateways, network policies, environments и dependency versions. Security tests запускаются только в разрешённом scope и не должны повреждать общие данные.

28. CORS (что это и в общих чертах механизм работы)

   **Ответ:** CORS — browser mechanism, который разрешает server сообщить, каким origins доступен cross-origin response. Для некоторых requests browser сначала отправляет preflight `OPTIONS` с origin, method и headers, а server отвечает `Access-Control-Allow-*`; simple request может уйти без preflight. CORS не является server-to-server защитой и не заменяет Authentication или authorization.

29. Архитектура:

   **Ответ:** API test framework обычно разделяет transport client, models и serialization, domain services, data builders, assertions, configuration и reporting. Tests вызывают business operations, но сохраняют доступ к raw request и response. Общие retries и Authentication размещаются централизованно, сохраняя явные границы и диагностику.

30. контрактное тестирование (Pact, OpenAPI)

   **Ответ:** OpenAPI-based tests проверяют, соответствует ли implementation описанной schema и operations. Pact реализует consumer-driven contract: consumer публикует ожидаемые interactions, а provider подтверждает их на своей стороне. Contract tests быстро находят несовместимость между services, но не заменяют functional и E2E testing.

31. версионирование API

   **Ответ:** Версию можно передавать в URI, header или media type; главное — единая документированная политика. Backward-compatible additions обычно не требуют новой major version, а удаление или изменение semantics требует controlled migration. Tests должны запускаться для поддерживаемых versions и проверять deprecation headers или сроки отключения, если они используются.

32. pipeline запросов с передачей контекста между шагами

   **Ответ:** Один шаг создаёт resource и сохраняет только необходимые значения, например id или token, в scenario context, а следующий использует их. Context должен быть отдельным для каждого test и thread, чтобы parallel execution не смешивал данные. Teardown удаляет созданные resources, а logs связывают requests через correlation id.

33. Протоколы и форматы:

   **Ответ:** Protocol определяет правила взаимодействия, transport доставляет данные, а format описывает их представление. Например, HTTP может переносить JSON, SOAP использует XML message model, а WebSocket переносит application-defined text или binary messages. Эти понятия нужно разделять при выборе tools и assertions.

34. GraphQL: query, mutation, schema validation

   **Ответ:** GraphQL `query` читает данные, `mutation` выполняет изменения, а schema определяет types, fields, arguments и доступные operations. Response может иметь HTTP 200 и одновременно содержать массив `errors`, поэтому проверяются обе части GraphQL response. Validation включает соответствие query schema, types и business authorization каждого field.

35. WebSockets: работа через брокеры, pub/sub

   **Ответ:** WebSocket может быть каналом между client и gateway, а broker реализует topics, routing и pub/sub за gateway. Это разные уровни: WebSocket не предоставляет broker semantics сам по себе. В тесте проверяются subscription, delivery, filtering, duplicates, ordering и reconnect с учётом гарантий конкретного broker.

36. SOAP + WS-Security

   **Ответ:** WS-Security добавляет security information в SOAP Header: signatures, encryption, timestamps и security tokens. Тест проверяет корректный namespace, signature, certificate, expiry и поведение при изменённом или отсутствующем security header. Transport TLS всё равно нужен, потому что message-level security и channel security решают разные задачи.

37. Безопасность и контроль:

   **Ответ:** Контроль включает least privilege, schema и input validation, audit trail, rate limits, inventory API versions и управление secrets. Негативные tests проверяют чужие object ids, запрещённые fields и functions, malformed data и утечку информации. Результаты security testing обрабатываются конфиденциально.

38. OWASP API security

   **Ответ:** OWASP API Security Top 10 помогает системно проверять риски вроде Broken Object Level Authorization, Broken Authentication, unrestricted resource consumption, security misconfiguration и unsafe consumption of APIs. Я использую список как checklist для threat modeling и test design, но не как полную security methodology. Приоритет зависит от architecture и business impact конкретного API.

39. тестирование ролей, токенов, JWT

   **Ответ:** Для каждой operation строится матрица role → разрешённое действие и проверяются положительные и отрицательные случаи, включая доступ к чужому object id. Для token проверяются issuer, audience, expiry, scope и revocation policy; для signed JWT также signature, algorithm и key id. Простое декодирование JWT не доказывает его подлинность.

40. Rate limiting, retries

   **Ответ:** Rate limiting проверяется около границы: допустимые requests проходят, превышение даёт согласованный status, часто 429, и headers вроде `Retry-After`, если они предусмотрены. Retry выполняют с ограничением attempts, exponential backoff и jitter. Автоматически повторять безопасно только идемпотентную operation или request с отдельной защитой от duplicate side effects.

41. Инфраструктура:

   **Ответ:** API tests зависят от endpoints, DNS, certificates, gateways, service discovery, databases, queues и observability. Эти зависимости должны иметь versioned configuration, health checks и понятный ownership. Test framework не должен скрывать infrastructure failure как обычный assertion failure.

42. интеграция в CI/CD

   **Ответ:** В Pull Request запускаются быстрые contract и component/API tests, после deployment — Smoke, а широкий набор — по согласованному этапу или расписанию. Pipeline передаёт environment и secrets защищённо, публикует requests/responses с маскированием и возвращает корректный exit code. Нестабильный внешний service лучше заменить controlled mock на раннем этапе.

43. multi-env execution

   **Ответ:** Код tests остаётся одинаковым, а base URL, credentials, certificates, feature flags и timeouts приходят из environment-specific configuration. Test data и expected capabilities окружения тоже должны быть явными. Запрещённые destructive tests нельзя случайно запустить в production, поэтому environments защищаются allowlist и отдельными credentials.

44. API mocking, virtual services

   **Ответ:** Mock или virtual service возвращает управляемые responses внешней dependency и позволяет проверить errors, timeouts и редкие состояния. Он должен соответствовать contract и регулярно проверяться против реального provider, иначе появляется ложная уверенность. Для критичных integrations нужны и contract tests, и ограниченный набор tests с настоящим service.
