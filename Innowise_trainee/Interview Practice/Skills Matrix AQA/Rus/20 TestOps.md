# TestOps

## Содержание

- [[#Вопросы и ответы]]

**Связанные заметки:** [[00 Индекс Skills Matrix AQA]]

---

## Вопросы и ответы

1. Понимание концепции TestOps и ее роли в жизненном цикле разработки ПО (DevOps/DevTestOps).

   **Ответ:** TestOps объединяет planning, test automation, environments, data, execution, results и analytics в управляемый continuous process. Его цель — быстро давать команде достоверную информацию о качестве на всём delivery lifecycle. Это не один инструмент, а совместная практика QA, development, DevOps и product roles.

2. Базовое представление об одной из облачных платформ (AWS, Azure, или GCP):

   **Ответ:** Для одной платформы я должен понимать regions/zones, compute, storage, networking, IAM, monitoring и billing. На собеседовании лучше выбрать реально изученную cloud platform и связать services с конкретной test task. Названия services важны, но ещё важнее lifecycle resource, права и стоимость.

3. Знание основных сервисов, релевантных для тестирования (e.g., EC2/VMs, S3/Blob Storage, basic IAM/User Management).

   **Ответ:** Compute service запускает VM или workload, object storage хранит test data и artifacts, IAM управляет identities и permissions. Дополнительно часто нужны virtual network, load balancer, logs, metrics, database, container registry и managed Kubernetes. Для каждого service тестировщик должен понимать доступ, cleanup, encryption и cost model хотя бы на базовом уровне.

4. Умение заходить в консоль облака и выполнять простые задачи под руководством (e.g., запуск/остановка VM, просмотр логов).

   **Ответ:** Я выбираю правильные account, project/subscription и region, проверяю tags и permissions, затем выполняю разрешённую операцию и контролирую status. Перед остановкой VM учитываю влияние на shared environment и persistent data. Все ручные действия лучше фиксировать, а повторяемые операции постепенно переводить в IaC.

5. Понимание, зачем тестовые окружения могут разворачиваться в облаке.

   **Ответ:** Cloud позволяет быстро создавать environment нужного размера, масштабировать parallel tests и удалять resources после использования. Managed services помогают приблизить configuration к production и выполнять cross-region или resilience scenarios. Компромиссы — стоимость, security, quotas, startup time и риск configuration drift.

6. Знание, что автотесты могут интегрироваться с CI/CD пайплайнами, которые взаимодействуют с облачными ресурсами.

   **Ответ:** Pipeline может через IaC создать environment, получить endpoints, развернуть build, запустить tests, собрать artifacts и удалить resources. Доступ выдаётся short-lived identity или service role с минимальными permissions. Cleanup должен выполняться даже после failed test, иначе остаются дорогие и небезопасные resources.

7. Практический опыт работы с одной или несколькими облачными платформами (AWS, Azure, GCP) для задач тестирования:

   **Ответ:** Здесь нужен реальный пример: «Я использовал `[cloud]` для `[задача]`, работал с `[реальные services]` и отвечал за `[ваши действия]`». Важно назвать способ доступа, deployment, наблюдение и cleanup. Учебный sandbox следует честно обозначить как учебный, не добавляя вымышленный production опыт.

8. Умение самостоятельно разворачивать и настраивать тестовые окружения (e.g., создание VMs/контейнеров, настройка security groups/NSGs, базовых сетевых конфигураций).

   **Ответ:** Я определяю network, subnets, routes, DNS и минимально необходимые inbound/outbound rules, затем создаю compute или containers и применяю configuration. Доступ ограничивается source, port и identity, а public exposure используется только при необходимости. Readiness, logs, resource tags, expiration и cleanup закладываются сразу.

9. Опыт работы с сервисами хранения данных (e.g., S3, Azure Blob Storage, Google Cloud Storage) для тестовых данных или артефактов.

   **Ответ:** Object storage подходит для test datasets, screenshots, videos, reports и build artifacts. Я настраиваю bucket/container permissions, encryption, versioning или lifecycle/retention и передаю ссылку с ограниченным доступом. Sensitive data требует classification, masking и запрета public access.

10. Понимание и применение IAM ролей/политик для безопасного доступа к облачным ресурсам для тестовых нужд.

   **Ответ:** IAM policy выдаёт identity только actions над нужными resources и на нужный срок. Для CI предпочтительны workload identity и short-lived credentials вместо постоянных access keys. Я проверяю negative access, разделяю human и service roles и регулярно удаляю неиспользуемые permissions.

11. Опыт интеграции автотестов с CI/CD системами (e.g., Jenkins, GitLab CI, Azure DevOps, GitHub Actions) для запуска тестов в облачных окружениях.

   **Ответ:** Ответ можно построить так: «Pipeline в `[система]` получал cloud identity, создавал или выбирал environment, запускал `[tests]`, публиковал `[artifacts]` и выполнял cleanup». Следует объяснить trigger, quality gate и обработку failure. Если опыт учебный, нужно явно указать это.

12. Умение использовать инструменты Infrastructure as Code (IaC) на базовом уровне (e.g., Terraform, CloudFormation, ARM templates, Google Cloud Deployment Manager) для создания или управления тестовыми окружениями.

   **Ответ:** IaC декларативно описывает resources, позволяет review изменений, повторяемое создание и controlled destroy. Базовый процесс — format/validate, plan, approval, apply и сохранение защищённого state. Google Cloud Deployment Manager из исходного списка уже снят с поддержки 31 марта 2026 года; для новых решений следует выбирать Infrastructure Manager с Terraform или другую поддерживаемую технологию.

13. Навыки мониторинга и сбора базовых метрик тестовых окружений в облаке.

   **Ответ:** Я собираю CPU, memory, disk, network, request rate, latency, error rate и health, а logs связываю через timestamp и correlation id. Dashboard показывает состояние environment во время tests, а alert сообщает о реальной проблеме, не создавая шум. Retention и доступ к telemetry соответствуют sensitivity данных и стоимости.

14. Понимание концепций масштабируемости и отказоустойчивости в облаке и как они могут применяться к тестовым окружениям.

   **Ответ:** Scalability означает изменение capacity под load, а fault tolerance — сохранение работы при отказе части системы. В test environment можно проверить autoscaling, распределение по zones, health-based replacement и восстановление dependency. Масштаб среды должен соответствовать цели теста: маленькая копия не подтверждает production capacity.

15. Глубокий практический опыт проектирования, внедрения и управления TestOps стратегиями и процессами.

   **Ответ:** Этот пункт требует реального кейса: `[исходная проблема] → [стратегия environments/data/execution/reporting] → [ваша роль] → [подтверждённый результат]`. Нужно объяснить governance, ownership, metrics и улучшение по feedback. Без такого опыта лучше честно описать предлагаемую стратегию и учебную реализацию.

16. Продвинутые навыки работы с несколькими облачными платформами (AWS, Azure, GCP):

   **Ответ:** Multi-cloud опыт означает понимание не только аналогов services, но и различий IAM, networking, observability, quotas и billing. Общий IaC/module может унифицировать intent, но platform-specific capabilities остаются в adapters. Такой подход оправдан business requirements, а не желанием использовать больше технологий.

17. Проектирование и реализация сложных, масштабируемых и безопасных тестовых окружений в облаке с использованием лучших практик.

   **Ответ:** Я начинаю с threat model, network boundaries, data classification, capacity и lifecycle, затем проектирую immutable deployment через IaC. Используются private networking, least privilege, managed secrets, encryption, policies, backups и observability. Environment автоматически проверяется, маркируется owner/TTL и безопасно удаляется.

18. Опыт работы с контейнеризацией и оркестрацией (e.g., Docker, Kubernetes - EKS, AKS, GKE) для динамического управления тестовыми окружениями.

   **Ответ:** Managed Kubernetes может создавать namespace или ephemeral release на branch, запускать application и tests и удалять их после pipeline. Images фиксируются digest, workloads имеют resources и probes, а access ограничивается service account и network policy. Реальный ответ должен указать `[ваша платформа, objects и ответственность]` без вымышленного опыта.

19. Продвинутое использование сервисов баз данных (e.g., RDS, Azure SQL, Cloud SQL), очередей сообщений, serverless функций (Lambda, Azure Functions, Cloud Functions) для создания реалистичных тестовых сценариев.

   **Ответ:** Managed database даёт реалистичный engine и backup behavior, queue — asynchronous delivery semantics, а serverless function — event-driven processing и scaling. Tests управляют schema/data, correlation ids, eventual consistency и cleanup, не полагаясь на порядок случайных сообщений. Стоимость, quotas и cold start учитываются в design и metrics.

20. Экспертиза в области Infrastructure as Code (IaC) (e.g., Terraform, Pulumi, CDK) для полной автоматизации создания, обновления и удаления тестовой инфраструктуры.

   **Ответ:** Экспертиза включает modules, remote state и locking, policy checks, secrets, drift detection, imports, migrations и безопасный destroy. Pipeline формирует plan, требует review для рискованных изменений и применяет его под отдельной identity. Ephemeral environments получают уникальные names и TTL, а cleanup контролируется независимо от результата tests.

21. Разработка и внедрение стратегий мониторинга, логирования и оповещения для тестовых окружений и CI/CD пайплайнов (e.g., CloudWatch, Azure Monitor, Google Cloud's operations suite, ELK stack, Prometheus, Grafana).

   **Ответ:** Стратегия объединяет infrastructure, application и pipeline signals: metrics, logs, traces и events с общей metadata build/environment. Dashboards отвечают на конкретные вопросы, а alerts основаны на symptoms и SLO, имеют owner и runbook. Secrets и персональные данные маскируются до отправки в central storage.

22. Оптимизация затрат на облачные ресурсы, используемые для тестирования.

   **Ответ:** Я добавляю tags owner/project/TTL, schedules остановки, automatic cleanup, right-sizing, autoscaling и budgets/alerts. Ephemeral environments существуют только во время работы, а storage имеет lifecycle policy. Spot/preemptible capacity подходит для retryable workloads, но не для test, где внезапное прерывание исказит результат.

23. Проектирование и реализация CI/CD пайплайнов с фокусом на скорость, надежность и эффективность тестирования в облаке.

   **Ответ:** Pipeline выполняет быстрые static/unit/contract stages раньше, использует cache и parallelism, затем создаёт environment и запускает impacted tests. Retries применяются только к известным infrastructure transients, а test failures остаются видимыми. Reusable templates, immutable artifacts, observability и guaranteed cleanup повышают надёжность без скрытия проблем.

24. Менторство и обучение других членов команды по вопросам TestOps и использования облачных технологий.

   **Ответ:** Я бы начал с безопасного sandbox, коротких labs по IAM, deployment, logs и cost, затем дал участнику реальную небольшую задачу с review. Runbooks и examples фиксируют общие операции, а pair work помогает разобрать incidents. Результат обучения оценивается по самостоятельному безопасному выполнению, а не по числу презентаций.

25. Способность оценивать и выбирать подходящие облачные сервисы и инструменты для решения конкретных задач тестирования.

   **Ответ:** Я сравниваю functional fit, security/compliance, integration, skills, reliability, quotas, lock-in и total cost. Например, managed database ускоряет setup, но может быть дороже и отличаться от production configuration; container на VM даёт контроль, но требует обслуживания. Решение подтверждается small proof of concept и измеримыми критериями.

26. Опыт работы с вопросами безопасности в облаке (e.g., управление секретами, сетевая безопасность, соответствие стандартам) применительно к тестовым данным и окружениям.

   **Ответ:** Secrets хранятся в managed secret service и выдаются workload identity, network строится private-by-default, а access и changes audit-ятся. Test data классифицируется, маскируется и удаляется по retention policy; production PII не копируется без разрешённого процесса. Реальный ответ должен привести только подтверждённый пример `[ваша задача безопасности]`.

27. Наличие релевантных облачных сертификаций (e.g., AWS Certified Solutions Architect, Azure Administrator/DevOps Engineer, Google Professional Cloud DevOps Engineer) является плюсом и подтверждением знаний.

   **Ответ:** Сертификация подтверждает, что кандидат изучил определённый scope и сдал экзамен, но не заменяет практический опыт проектирования и диагностики. На собеседовании я бы назвал только действующую реальную сертификацию и показал, как применял знания. Если сертификата нет, можно подтвердить уровень repository, lab и конкретным разбором решений.
