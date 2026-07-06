# Tasks: add-request-withdrawal

## 1. Discovery and Specification

- [x] 1.1 Подтвердить текущее поведение через API, UI и внутренний инструмент поддержки
- [x] 1.2 Зафиксировать race между отзывом и стартом согласования
- [x] 1.3 Подготовить proposal со scope, non-goals и platform boundary
- [x] 1.4 Подготовить delta spec
- [x] 1.5 Согласовать permission code `approval-request.withdraw-own`
- [x] 1.6 Провести review Scenarios вместе с аналитиком и QA

## 2. Design

- [x] 2.1 Описать domain validation и error mapping
- [x] 2.2 Описать optimistic concurrency и atomic transition
- [x] 2.3 Описать API contract
- [x] 2.4 Разделить application logic, конфигурация приложения и возможности платформы
- [x] 2.5 Описать transactional outbox и Kafka delivery semantics
- [x] 2.6 Определить audit event type и allowlist attributes
- [x] 2.7 Подготовить rollout, rollback и observability
- [x] 2.8 Провести architecture и platform integration review
- [x] 2.9 Принять решение по ADR

## 3. Application Configuration

- [x] 3.1 Добавить permission `approval-request.withdraw-own`
- [x] 3.2 Добавить role mappings для `employee` и `manager`
- [x] 3.3 Зарегистрировать event type `APPROVAL_REQUEST_WITHDRAWN`
- [x] 3.4 Добавить topic alias `platform-audit`
- [x] 3.5 Связать topic name с `${PLATFORM_AUDIT_TOPIC}`
- [x] 3.6 Добавить schema version и allowlist audit attributes
- [x] 3.7 Проверить TEST и PROD topic ACL с platform team

## 4. Implementation

- [x] 4.1 Добавить `RequestWithdrawalService`
- [x] 4.2 Добавить transition `Submitted → Withdrawn`
- [x] 4.3 Запретить transition после approval started marker
- [x] 4.4 Добавить optimistic version check
- [x] 4.5 Добавить endpoint `POST /requests/{requestId}/withdraw`
- [x] 4.6 Добавить mapping для `403`, `404`, `409`, `412`
- [x] 4.7 Добавить атомарную запись audit event в platform outbox
- [x] 4.8 Добавить correlation ID propagation
- [x] 4.9 Добавить UI action и confirmation dialog
- [x] 4.10 Добавить UI mapping business errors
- [x] 4.11 Добавить structured application log без чувствительных данных

## 5. Audit Contract

- [x] 5.1 Добавить JSON schema `approval-request-withdrawn-v1`
- [x] 5.2 Проверить обязательные envelope fields
- [x] 5.3 Запретить дополнительные attributes через schema
- [x] 5.4 Проверить отсутствие содержимого заявки, token и PII
- [x] 5.5 Проверить стабильность `eventId` при platform retry
- [x] 5.6 Проверить публикацию в TEST Kafka topic
- [x] 5.7 Проверить platform metrics и поиск event по correlation ID

## 6. Automated Testing

- [x] 6.1 Добавить unit tests ownership и permission-independent domain rules
- [x] 6.2 Добавить unit tests allowed status и approval marker
- [x] 6.3 Добавить unit test audit payload mapping
- [x] 6.4 Добавить integration test atomic state update + outbox insert
- [x] 6.5 Добавить integration test rollback при ошибке outbox insert
- [x] 6.6 Добавить concurrency test отзыва и старта согласования
- [x] 6.7 Добавить API tests для `200`, `403`, `404`, `409`, `412`
- [x] 6.8 Добавить contract test audit schema version `1`
- [x] 6.9 Выполнить regression tests отправки и согласования

## 7. Manual QA

- [x] 7.1 Проверить успешный отзыв автором
- [x] 7.2 Проверить пользователя без permission
- [x] 7.3 Проверить попытку отзыва чужой заявки
- [x] 7.4 Проверить отзыв после старта согласования
- [x] 7.5 Проверить повторный запрос
- [x] 7.6 Проверить UI visibility и обновление после `409`
- [x] 7.7 Найти audit event по request ID и correlation ID
- [x] 7.8 Проверить Kafka outage на TEST и последующую retry delivery
- [x] 7.9 Получить QA sign-off

## 8. Review, Design Harvest and Archive

- [x] 8.1 Создать PR со ссылкой на OpenSpec change
- [x] 8.2 Устранить review comments
- [x] 8.3 Получить PR approval
- [x] 8.4 Выполнить Design Harvest
- [x] 8.5 Создать ADR-001 для status transition rule
- [x] 8.6 Подтвердить, что platform audit integration не требует отдельного ADR
- [x] 8.7 Обновить ADR traceability index
- [x] 8.8 Проверить соответствие proposal, spec, design, code и tests
- [x] 8.9 Выполнить archive после production rollout и verification
