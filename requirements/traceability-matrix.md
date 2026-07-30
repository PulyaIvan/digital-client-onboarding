# Матрица трассируемости

| User Story | Функциональные требования | Диаграммы |
|---|---|---|
| US-001 | FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, FR-007, FR-008 | erd/schema.dbml, sequence/*.puml, state/*.puml |
| US-002 | FR-009 | нет (TODO) |

FR-008 (Access Recovery) — см. [ADR-006](../decisions/adr-006-access-recovery-otp.md), эндпоинты `POST /application/{applicationId}/access-recovery` и `.../confirm` в `api/openapi.yaml`; отдельной диаграммы пока нет (TODO).

FR-009 (очередь менеджера + история) — `GET /application` (список, фильтр по `status`) и поле `history` в `ApplicationDetailsResponse`, `api/openapi.yaml`; `history` уже опирается на существующую таблицу `application_history` в `database/schema.sql` (новых таблиц не требует). Диаграмм пока нет — см. [sync-debt.md](sync-debt.md).

## TODO — заполнить ссылками на конкретные файлы диаграмм по мере создания
