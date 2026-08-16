# Матрица трассируемости

| User Story | Функциональные требования | Диаграммы |
|---|---|---|
| US-001 | FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, FR-007, FR-008 | erd/ERD.dbml, sequence/*.puml, state/*.puml |
| US-002 | FR-009 | нет — не пробел, см. ниже |

FR-008 (Access Recovery) — см. [ADR-006](../decisions/adr-006-access-recovery-otp.md), эндпоинты `POST /application/{applicationId}/access-recovery` и `.../confirm` в `api/openapi.yaml`; диаграмма — [sequence/Диаграмма последовательности восстановление access_token.puml](../diagrams/sequence/Диаграмма%20последовательности%20восстановление%20access_token.puml).

Донос данных клиентом (`POST .../additional-info`, `POST .../documents`, часть ADR-003) — диаграмма
[sequence/Диаграмма последовательности донесение данных клиентом.puml](../diagrams/sequence/Диаграмма%20последовательности%20донесение%20данных%20клиентом.puml).
Автопереход в `EXPIRED` батч-джобом (см. [ADR-003](../decisions/adr-003-additional-info-request-flow.md),
раздел «Решение») на диаграмме не показан — отдельного визуального представления для этого шага пока нет.

FR-009 (очередь менеджера + история) — `GET /application` (список, фильтр по `status`), `GET /application/{applicationId}` и поле `history` в `ApplicationDetailsResponse`, `api/openapi.yaml`; `history` уже опирается на существующую таблицу `application_history` в `database/schema.sql` (новых таблиц не требует). Диаграммы сознательно нет: это read-only запросы внутри одного сервиса без координации с другими системами, а sequence-диаграммы в этом проекте документируют межсистемное взаимодействие, а не путь пользователя по экранам — показывать сам `GET`-запрос было бы так же избыточно, как рисовать, что клиент листает форму заявки.

## Нефункциональные требования → артефакты

| НФТ | Артефакты, реализующие требование |
|---|---|
| НФТ-001 (Throughput) | Offset-пагинация `GET /application` (`limit`/`offset`, `api/openapi.yaml`) спроектирована с учётом расчётной нагрузки |
| НФТ-002 (Storage) | Документы хранятся по ссылке (`storage_ref`) в отдельном vault, а не в БД — `applicant_documents` (`database/schema.sql`), Applicant Document / Vault в [глоссарии](glossary.md) |
| НФТ-003 (Latency) | Асинхронная обработка KYC — [ADR-001](../decisions/adr-001-async-kyc-processing.md); `202 Accepted` на всех асинхронных операциях (`api/openapi.yaml`) |
| НФТ-004 (Uptime) | Онлайн-приём заявок 24/7 — [business/goals.md](../business/goals.md); отдельных архитектурных артефактов (резервирование, failover) в этом пет-проекте не заводилось |
| НФТ-005 (Security) | Шифрование ПДн — `applicant_sensitive_data` (`*_encrypted`, `database/schema.sql`); ролевой доступ — `BearerAuth`/`AccessTokenAuth`/`KycServiceAuth` (`api/openapi.yaml`, [ADR-005](../decisions/adr-005-access-token-client-endpoints.md), [ADR-008](../decisions/adr-008-pii-in-kafka-events.md)) |
