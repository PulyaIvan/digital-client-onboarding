# Матрица трассируемости

| User Story | Функциональные требования | Диаграммы |
|---|---|---|
| US-001 | FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, FR-007, FR-008 | erd/ERD.dbml, sequence/*.puml, state/*.puml |
| US-002 | FR-009 | нет — не пробел, см. ниже |

FR-008 (Access Recovery) — см. [ADR-006](../decisions/adr-006-access-recovery-otp.md), эндпоинты `POST /application/{applicationId}/access-recovery` и `.../confirm` в `api/openapi.yaml`; диаграмма — [sequence/Диаграмма последовательности восстановление access_token.puml](../diagrams/sequence/Диаграмма%20последовательности%20восстановление%20access_token.puml).

Донос данных клиентом (`POST .../additional-info`, `POST .../documents`, часть ADR-003) — диаграмма
[sequence/Диаграмма последовательности донесение данных клиентом.puml](../diagrams/sequence/Диаграмма%20последовательности%20донесение%20данных%20клиентом.puml).
Автопереход в `EXPIRED` батч-джобом (см. [[project_expired_batch_job]] в памяти проекта / ADR-003
"Решение") на диаграмме не показан — отдельного визуального представления для этого шага пока нет.

FR-009 (очередь менеджера + история) — `GET /application` (список, фильтр по `status`), `GET /application/{applicationId}` и поле `history` в `ApplicationDetailsResponse`, `api/openapi.yaml`; `history` уже опирается на существующую таблицу `application_history` в `database/schema.sql` (новых таблиц не требует). Диаграммы сознательно нет — это read-only запросы внутри одного сервиса без межсистемной координации, sequence-диаграммы такие шаги принципиально не показывают (см. `requirements/sync-debt.md`, раздел «Не пробел, а осознанный скоуп»).
