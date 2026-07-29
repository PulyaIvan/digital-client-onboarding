# Глоссарий


| Applicant | Физическое лицо, подающее заявку на открытие счёта |
| Application | Заявка на открытие счёта, привязанная к Applicant |
| Application status | Статус заявки (`application.status`): CREATED / PENDING / APPROVED / REJECTED / MANUAL_REVIEW / ADDITIONAL_INFO_REQUIRED / EXPIRED / COMPLETED. Не путать с KYC verdict — статус общий для решения и от KYC, и от менеджера; кто именно принял решение, фиксируется в `application_history.changed_by`. Модель переходов и обоснование новых статусов (ADDITIONAL_INFO_REQUIRED / EXPIRED) — см. [ADR-003](../decisions/adr-003-additional-info-request-flow.md) |
| Additional Info Request | Запрос на повторное предоставление части данных/документов по заявке (статус `ADDITIONAL_INFO_REQUIRED`), инициируется системой (из PENDING) или менеджером (из MANUAL_REVIEW), ограничен сроком `additionalInfoDeadline` — см. [ADR-003](../decisions/adr-003-additional-info-request-flow.md) |
| KYC verdict | Итоговое решение KYC-провайдера (`kyc_result.result`): approved / rejected / manual_review |
| Manual Review | Промежуточный статус заявки, при котором решение принимает не KYC-провайдер, а менеджер вручную через административный интерфейс |
| Applicant Document | Скан/фото документа клиента (паспорт, фото), хранится по ссылке в защищённом хранилище (vault), не в основной БД |
| Sensitive Data | Чувствительные персональные данные клиента (паспорт, СНИЛС, ИНН, адреса), хранятся в БД в зашифрованном виде |
| Onboarding Service | Внутренний сервис, принимающий заявку, хранящий Applicant/Application и оркестрирующий процесс через Kafka |
| Core Banking | Центральная система учёта счетов, балансов и транзакций банка; создаёт Debit_account после одобрения заявки |
| Notification Service | Внутренний сервис, отправляющий клиенту уведомления (push/email/sms) о статусе заявки |
| Kafka (Event Bus) | Брокер сообщений для асинхронного взаимодействия между Onboarding Service, Core Banking и Notification Service |

## TODO — дополнить по мере проработки
