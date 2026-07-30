# Расхождения: API/ADR ↔ БД и диаграммы

Зафиксировано по состоянию на ADR-004/005/006. Осознанный выбор: сначала полностью закрыть
вопросы на уровне `api/openapi.yaml`/ADR, не отвлекаясь на `database/schema.sql` и
`diagrams/`, и затем одним проходом свести их под уже стабилизировавшийся контракт — чтобы не
перерисовывать диаграммы при каждой правке API. Этот файл — конкретный список того, что именно
нужно будет свести, когда дойдёт очередь.

## database/schema.sql + diagrams/erd/ERD.dbml

Три таблицы существуют только на уровне API-контракта, в БД/ERD не заведены:
- `idempotency_keys` (key, request_hash, cached_response, created_at, expires_at) — ADR-004.
- Хранение `accessToken` — ни колонки, ни отдельной таблицы нет; нужен хеш, а не открытый
  текст (по аналогии с паролем) — ADR-005.
- `access_recovery_code` (application_id, code_hash, expires_at, attempts, used_at) — ADR-006.

Несоответствия в уже существующих таблицах:
- `application_history.changed_by varchar(50)` — комментарий "заполняется только для решений
  менеджера в MANUAL_REVIEW" описывает старую модель, где `changedBy` был полем в теле запроса
  (`ApplicationApproveRequest`/`ApplicationRejectRequest`/`ApplicationRequestAdditionalInfoRequest`).
  Это поле убрано из `api/openapi.yaml` — личность менеджера теперь должна браться из
  `BearerAuth` (JWT), а не из тела запроса. Комментарий к колонке всё ещё описывает старый
  источник значения.
- `doc_type_enum('passport', 'photo')` — в API те же сущности называются
  `passportScan`/`photoApplicant` (в т.ч. в новом `GET /application/{applicationId}/documents/{documentType}`).
  Само по себе разное именование в API/БД — не проблема, но сопоставление `passport ↔
  passportScan`, `photo ↔ photoApplicant` нигде явно не зафиксировано.
- `application_status_enum` (БД/ERD) содержит `CREATED` и `COMPLETED`, которых нет в
  `ApplicationStatusResponse.status` (`api/openapi.yaml`: `PENDING`, `MANUAL_REVIEW`,
  `ADDITIONAL_INFO_REQUIRED`, `APPROVED`, `REJECTED`, `EXPIRED`). Отсутствие `CREATED` в API
  оправдано (внутренний статус до отправки в KYC, клиент его не видит — синхронный ответ уже
  возвращает `PENDING`). Отсутствие `COMPLETED` выглядит как реальный пробел, а не намеренное
  решение: клиент, поллящий `GET /status`, в принципе никогда не увидит подтверждение, что счёт
  открыт. Не связано с сегодняшними ADR — обнаружено при сверке, требует отдельного решения.

## diagrams/sequence/*.puml

- `Диаграмма последовательно онбординг клиента (happy path).puml`: `cl -> svc: POST /application`
  не показывает заголовок `Idempotency-key`; `svc --> cl: 202 Accepted\nstatus=PENDING` не
  показывает выдачу `accessToken` в ответе (ADR-004, ADR-005).
- `Диаграмма последовательности(MANUAL_REVIEW).puml`: шаг
  `mgr -> svc: POST /application/{id}/approve\n{changed_by: managerId}` устарел дважды —
  (1) `changed_by` в теле запроса больше нет в API; (2) не показан обязательный теперь
  `BearerAuth`. Отдельно (не сегодняшнее расхождение, а изначально неполное покрытие
  сценария из ADR-003): в этой диаграмме нет ветки `request-additional-info`, только `approve`.
- Ни одна из sequence-диаграмм пока не показывает вообще: `GET /application/{id}/status` +
  `POST .../additional-info`/`POST .../documents` с `Access-Token`; новый
  `GET /application/{applicationId}` (менеджер смотрит полные данные заявки);
  `GET /application/{applicationId}/documents/{documentType}` (скачивание документа
  менеджером); `POST .../access-recovery` + `.../confirm` (ADR-006).

## X-Correlation-Id (наблюдаемость)

`api/openapi.yaml` теперь принимает необязательный заголовок `X-Correlation-Id` на всех
эндпоинтах и возвращает его во всех ответах (см. [non-functional.md](non-functional.md)).
Ни один sequence-диаграммы, ни один участник (Kafka event payload) этого пока не отражает —
события в шинах публикуются только с `{applicationId, ...}`, без correlation id конкретного
HTTP-вызова, инициировавшего цепочку. Дорисовать вместе с остальным пунктом sequence-диаграмм
выше.

## GET /application (список) и history (FR-009)

Новый `GET /application` (список заявок с фильтром по `status`, пагинация `limit`/`offset`) и
новое поле `history` в `ApplicationDetailsResponse` (`ApplicationHistoryEntry[]`) — ни то, ни
другое пока не в sequence-диаграммах (см. выше — общий список отсутствующих сценариев). Отдельно:
`history` **не требует** новой таблицы в БД — данные уже есть в `application_history`
(`database/schema.sql`), эндпоинт их просто ещё не читал. Единственное, что стоит иметь в виду:
`ApplicationHistoryEntry.status` намеренно использует полный enum из 8 статусов (включая
`CREATED`/`COMPLETED`), в отличие от 6-значного `ApplicationStatusResponse.status` — это разное
назначение полей (полная история vs live-статус), не рассинхрон между ними, но подсвечивает уже
описанный выше пробел с отсутствием `COMPLETED` в live-статусе ещё раз.

## /v1 (версионирование, ADR-007)

Все пути в `api/openapi.yaml` теперь под префиксом `/v1` (`/v1/application`, ...). Ни
sequence-диаграммы, ни `use-cases.md`/ADR-001–006 этот префикс не используют — они называют
эндпоинты в сокращённой форме (`POST /application`) осознанно, см. ADR-007 "Решение". Отдельного
пункта на дорисовку не требуется — это не рассинхрон, а принятая конвенция сокращения; `api/openapi.yaml`
остаётся источником буквального пути.

## diagrams/state/*.puml

- Диаграмма состояний `Application` авторизации не касается, расхождений с ADR-004/005/006 нет.
  Единственное несовпадение — то же, что и в БД: `COMPLETED` есть на диаграмме, но не в
  `ApplicationStatusResponse.status` API (см. выше) — общий пробел, не отдельная проблема
  диаграммы.
