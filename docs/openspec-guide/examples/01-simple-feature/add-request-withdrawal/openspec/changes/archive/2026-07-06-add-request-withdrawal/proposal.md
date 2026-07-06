# Proposal: add-request-withdrawal

## Why

После отправки заявки автор не может исправить ошибку самостоятельно. Даже если согласование ещё не началось, ему приходится обращаться в службу поддержки, а сотрудник поддержки вручную переводит заявку в `Withdrawn`.

Это создаёт несколько проблем:

- пользователь не может быстро остановить ошибочную заявку;
- поддержка выполняет лишнюю ручную операцию;
- разные сотрудники поддержки используют разные способы изменения статуса;
- причина и инициатор отзыва не всегда попадают в audit log;
- прямое изменение статуса может обойти бизнес-правила workflow.

## Current Behavior

- Пользователь может отправить заявку и видит её текущий статус.
- После отправки action отзыва в UI отсутствует.
- Public API не предоставляет операцию отзыва.
- Поддержка может изменить статус через внутренний инструмент.
- Согласование может начаться асинхронно сразу после отправки.
- Корпоративная платформа предоставляет access control и audit module.
- Application configuration уже содержит role mappings и Kafka topics для других audit events.

## Goal

Дать автору заявки возможность безопасно отозвать её, пока согласование не началось, и сформировать стандартное audit event через платформенный механизм.

## What Changes

- Добавить permission `approval-request.withdraw-own`.
- Разрешить отзыв только автору заявки с этим permission.
- Разрешить отзыв только из статуса `Submitted`, пока согласование не началось.
- Выполнять переход `Submitted → Withdrawn` атомарно через `RequestStatusTransitionService`.
- Добавить endpoint `POST /requests/{requestId}/withdraw`.
- Добавить action отзыва в UI при выполнении известных UI условий.
- После успешного перехода сформировать business audit event `APPROVAL_REQUEST_WITHDRAWN`.
- Передавать audit event стандартному platform audit module.
- Настроить topic и event mapping в конфигурация приложения.
- Добавить automated tests, manual QA и regression.

## Out of Scope

- Отзыв после начала согласования.
- Отзыв заявки другим пользователем, администратором или поддержкой.
- Массовый отзыв заявок.
- Изменение существующего процесса согласования.
- Реализация собственного Kafka producer, retry или outbox.
- Изменение platform audit module.
- Унификация всех старых прямых изменений статусов в рамках этого change.
- Добавление обязательного комментария или причины отзыва.

## Platform Boundary

Корпоративная платформа предоставляет:

- проверку permissions;
- audit event envelope;
- transactional outbox;
- сериализацию сообщения;
- публикацию в Kafka;
- retry и технические метрики доставки.

Приложение определяет:

- permission code и role mappings;
- момент создания business audit event;
- event type;
- entity type и entity ID;
- business attributes payload;
- topic mapping;
- данные, которые запрещено включать в audit event.

## Affected Areas

- Approval Request domain model.
- `RequestStatusTransitionService`.
- Request API.
- UI action availability.
- Access control configuration.
- Audit event configuration.
- Kafka topic configuration.
- Automated tests и test data.
- Monitoring dashboard для audit delivery.

## Risks and Constraints

- Согласование может стартовать одновременно с запросом отзыва.
- UI может показывать устаревший статус, поэтому backend validation обязательна.
- Duplicate HTTP request не должен создать повторный переход статуса.
- Platform audit delivery имеет семантику at-least-once; consumer должен использовать `eventId` для deduplication.
- В audit payload нельзя передавать полное содержимое заявки и чувствительные данные.
- Topic name отличается между environments и не должен быть hardcoded.
- Нельзя возвращать детали заявки пользователю без доступа к ней.

## Success Criteria

- Автор с permission отзывает собственную заявку до начала согласования.
- Пользователь без permission получает `403`.
- Пользователь не может отозвать чужую заявку.
- После начала согласования операция возвращает `409`, статус не меняется.
- Повторный отзыв возвращает `409`, повторный audit event не создаётся.
- Успешный переход и запись в platform outbox фиксируются в одной transaction.
- Platform audit module публикует `APPROVAL_REQUEST_WITHDRAWN` в настроенный Kafka topic.
- Audit event соответствует schema и не содержит запрещённых данных.
- Regression существующей отправки и согласования пройдена.

## Open Questions

Закрыты до review:

- Причина отзыва в первой версии не запрашивается.
- Для `manager` и `employee` permission выдаётся через application role mapping; `support` не получает его автоматически.
- Успешный API response возвращает обновлённые `status` и `version` заявки.
