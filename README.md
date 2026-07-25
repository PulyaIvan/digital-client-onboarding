# Digital Client Onboarding

Аналитическая документация процесса цифрового онбординга нового клиента банка: от подачи заявки до открытия дебетового счёта.

## О проекте

Автоматизированная система онбординга, которая:
- не требует участия человека в стандартном сценарии;
- поддерживает требования регулятора по идентификации клиента (KYC);
- масштабируется на тысячи заявок;
- предоставляет клиенту прозрачный статус на каждом этапе.

Скоуп ограничен сценарием онбординга нового клиента, в рамках которого верификация личности (KYC) и открытие первого счёта происходят как единый процесс. Подробнее — [business/goals.md](business/goals.md).

## Ключевые сущности процесса

- **Client** — новый клиент банка, подающий заявку
- **Application** — заявка на открытие счёта
- **KYC** — проверка личности клиента и оценка риска (approved / rejected / manual_review)
- **Core Banking** — система учёта счетов, балансов и транзакций
- **Notification Service** — уведомления клиента о статусе заявки
- **Kafka (Event Bus)** — асинхронное взаимодействие между сервисами

Заявка обрабатывается асинхронно: API отвечает `202 Accepted` сразу после валидации, а финальный статус KYC приходит через уведомление или поллинг статуса заявки. Счёт в Core Banking создаётся идемпотентно — ровно один раз на заявку. Обоснование — в [decisions/](decisions/).

## Структура репозитория

| Папка | Содержимое |
|---|---|
| [business/](business/) | Бизнес-цели и контекст проекта |
| [requirements/](requirements/) | Функциональные и нефункциональные требования, use cases, глоссарий, матрица трассируемости |
| [decisions/](decisions/) | ADR — архитектурные решения и их обоснование |
| [diagrams/](diagrams/) | C4, BPMN, ERD, sequence и state-диаграммы (исходники `.puml`/`.dbml` и экспорты в `exports/`) |
| [api/](api/) | OpenAPI-спецификация (в разработке) |
| [database/](database/) | Схема БД (в разработке) |

## Требования

- [Функциональные требования](requirements/functional.md) (FR-001 – FR-007)
- [Нефункциональные требования](requirements/non-functional.md)
- [Use cases и user stories](requirements/use-cases.md)
- [Глоссарий](requirements/glossary.md)
- [Матрица трассируемости](requirements/traceability-matrix.md)

## Архитектурные решения (ADR)

- [ADR-001: Асинхронная обработка KYC-проверки](decisions/adr-001-async-kyc-processing.md)
- [ADR-002: Идемпотентное создание счёта в Core Banking](decisions/adr-002-idempotent-account-creation.md)

## Диаграммы

- **C4:** [diagrams/c4/C4 Диаграмма.puml](diagrams/c4/C4%20Диаграмма.puml)
- **ERD:** [diagrams/erd/schema.dbml](diagrams/erd/schema.dbml)
- **Sequence:** happy path, manual review, reject — [diagrams/sequence/](diagrams/sequence/)
- **State:** Application, Debit Account — [diagrams/state/](diagrams/state/)
- **Экспорты (PNG):** [diagrams/exports/](diagrams/exports/)

## Статус

Проект в стадии проработки требований. Открытые TODO:
- [non-functional.md](requirements/non-functional.md) — throughput, раздельные latency-требования, шифрование ПДн
- [glossary.md](requirements/glossary.md) — дополнение терминов
- [traceability-matrix.md](requirements/traceability-matrix.md) — ссылки на конкретные файлы диаграмм
- `api/openapi.yaml`, `database/schema.sql` — спецификации ещё не заполнены
