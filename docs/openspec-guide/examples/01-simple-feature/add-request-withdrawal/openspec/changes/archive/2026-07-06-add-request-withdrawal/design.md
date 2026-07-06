# Design: add-request-withdrawal

## Summary

Добавляется операция самостоятельного отзыва заявки до начала согласования.

Business validation выполняется в domain layer. Изменение статуса и создание audit outbox record фиксируются атомарно. Доставка audit event в Kafka выполняется стандартным platform audit module.

## Current Behavior

- Public API не содержит операции отзыва.
- UI не показывает action отзыва.
- Поддержка может вручную изменить статус через внутренний инструмент.
- `RequestStatusTransitionService` уже используется основным workflow, но несколько legacy handlers изменяют статус напрямую.
- Access control выполняется platform authorization module по permission codes из конфигурация приложения.
- Platform audit module уже публикует другие application events через transactional outbox в Kafka.

## Design Goals

- Единое бизнес-правило для UI, API и будущих внутренних вызовов.
- Атомарное разрешение race между отзывом и стартом согласования.
- Backend authorization независимо от UI visibility.
- Надёжная регистрация audit event без собственного Kafka producer.
- Отсутствие чувствительных данных в audit payload.
- Совместимость с существующим workflow и API.

## Non-Goals

- Рефакторинг всех legacy status updates.
- Изменение platform audit module.
- Exactly-once доставка в Kafka.
- Синхронное ожидание Kafka acknowledgement в HTTP request.
- Отзыв от имени другого пользователя.

## Responsibility Boundary

### Application business logic

Приложение отвечает за:

- проверку ownership заявки;
- проверку разрешённого current status;
- проверку факта старта согласования;
- переход `Submitted → Withdrawn`;
- выбор business audit event;
- business attributes audit payload;
- API response и domain error mapping;
- UI visibility rule.

### Application-owned platform configuration

Приложение отвечает за:

- permission `approval-request.withdraw-own`;
- role-to-permission mappings;
- event type `APPROVAL_REQUEST_WITHDRAWN`;
- topic alias `platform-audit`;
- зависящий от environment topic name;
- schema version;
- список разрешённых payload attributes;
- retry policy reference, если platform standard позволяет выбирать профиль.

### Platform capabilities

Платформа отвечает за:

- получение identity текущего пользователя;
- вычисление permissions из role mappings;
- API platform authorization adapter;
- audit envelope;
- запись platform outbox record;
- сериализацию по audit schema;
- Kafka producer;
- retry и dead-letter handling;
- delivery metrics и technical logs;
- propagation correlation ID.

Приложение не создаёт собственный Kafka client и не реализует повторную доставку.

## Request Flow

```text
UI
→ POST /requests/{requestId}/withdraw
→ platform authentication
→ platform permission check: approval-request.withdraw-own
→ WithdrawRequestHandler
→ RequestWithdrawalService
→ RequestStatusTransitionService
→ database transaction
   ├── update request status and version
   └── PlatformAuditOutbox.append(APPROVAL_REQUEST_WITHDRAWN)
→ commit
→ HTTP 200

Асинхронно:
platform outbox worker
→ serialize audit envelope
→ publish to configured Kafka topic
→ mark outbox record delivered
```

## Domain Validation

`RequestWithdrawalService.withdraw(requestId, actorId)` выполняет проверки в следующем порядке:

1. Загрузить заявку с блокировкой или optimistic version.
2. Проверить доступ пользователя к самой заявке без раскрытия лишних данных.
3. Проверить, что actor является автором.
4. Проверить status `Submitted`.
5. Проверить, что approval process не зафиксировал старт.
6. Выполнить transition через `RequestStatusTransitionService`.
7. Добавить audit event в platform outbox.
8. Зафиксировать transaction.

Permission проверяется platform adapter до handler. Ownership остаётся domain rule и проверяется на backend.

## Concurrency

Старт согласования и отзыв могут выполняться одновременно.

Выбранный подход:

- request row содержит optimistic `version`;
- transition service проверяет current status и approval marker внутри transaction;
- и старт согласования, и отзыв используют один transition service;
- при конфликте только одна transaction завершается успешно;
- проигравшая операция повторно читает state и возвращает domain error.

Для отзыва после победившего старта согласования возвращается:

```text
HTTP 409
error.code = APPROVAL_ALREADY_STARTED
```

## API Contract

### Request

```http
POST /requests/8f52a87e-95ce-4c6c-a989-2e0f941ff728/withdraw
Authorization: Bearer <token>
X-Correlation-ID: 98b40c52-60ad-46f1-9448-ac98f4838365
If-Match: 7
```

Body отсутствует.

### Success Response

```json
{
  "id": "8f52a87e-95ce-4c6c-a989-2e0f941ff728",
  "status": "Withdrawn",
  "version": 8,
  "withdrawnAt": "2026-07-06T09:30:00Z"
}
```

### Errors

| HTTP | error.code | Условие |
|---|---|---|
| 403 | `REQUEST_WITHDRAWAL_FORBIDDEN` | Нет permission или пользователь не является автором |
| 404 | `REQUEST_NOT_FOUND` | Заявка недоступна пользователю или не существует |
| 409 | `APPROVAL_ALREADY_STARTED` | Согласование уже началось |
| 409 | `REQUEST_STATUS_NOT_WITHDRAWABLE` | Current status не допускает отзыв |
| 409 | `REQUEST_ALREADY_WITHDRAWN` | Заявка уже отозвана |
| 412 | `REQUEST_VERSION_MISMATCH` | `If-Match` не совпадает с current version |

403 и 404 mapping должен соответствовать существующей security policy, чтобы не раскрывать наличие чужой заявки.

## Access Control Configuration

Permission определяется приложением, а enforcement выполняется платформой.

Пример конфигурация приложения:

```yaml
permissions:
  approval-request.withdraw-own:
    description: Withdraw own approval request before approval starts

roleMappings:
  employee:
    - approval-request.withdraw-own
  manager:
    - approval-request.withdraw-own
```

UI получает effective permissions стандартным platform способом. Отсутствие action в UI не заменяет backend check.

## Platform Audit Integration

### Почему аудит является частью change

Успешный отзыв меняет business state. Для расследований необходимо знать:

- кто инициировал отзыв;
- какая заявка была изменена;
- когда произошло изменение;
- из какого статуса и в какой выполнен переход;
- к какому HTTP request относится событие.

Specification фиксирует обязательность observable audit behavior. Design фиксирует технический platform contract.

### Event Type

```text
APPROVAL_REQUEST_WITHDRAWN
```

Event type принадлежит приложению и регистрируется в конфигурация приложения.

### Topic Configuration

Topic name не hardcoded в production-код.

```yaml
platform:
  audit:
    events:
      APPROVAL_REQUEST_WITHDRAWN:
        topicAlias: platform-audit
        schemaVersion: 1
        deliveryProfile: standard-durable
    topics:
      platform-audit:
        name: ${PLATFORM_AUDIT_TOPIC}
```

Environment configuration:

```text
DEV:  company.audit.events.dev.v1
TEST: company.audit.events.test.v1
PROD: company.audit.events.v1
```

Репозиторий хранит alias и environment variable reference. Конкретные credentials и broker addresses предоставляет platform runtime; секреты в конфигурация приложения не хранятся.

### Audit Message

Пример сообщения в Kafka:

```json
{
  "eventId": "56b6be15-86c2-4cee-8f2a-dcb2aa846551",
  "eventType": "APPROVAL_REQUEST_WITHDRAWN",
  "schemaVersion": 1,
  "occurredAt": "2026-07-06T09:30:00Z",
  "correlationId": "98b40c52-60ad-46f1-9448-ac98f4838365",
  "actor": {
    "type": "user",
    "id": "2ac9d890-79c7-4b0d-95f2-a9015e0df63d"
  },
  "entity": {
    "type": "approval-request",
    "id": "8f52a87e-95ce-4c6c-a989-2e0f941ff728"
  },
  "action": "withdraw",
  "outcome": "success",
  "attributes": {
    "previousStatus": "Submitted",
    "newStatus": "Withdrawn",
    "requestVersion": 8
  }
}
```

Запрещено включать:

- полное содержимое заявки;
- access token;
- персональные данные, не необходимые для аудита;
- свободный текст пользователя;
- platform credentials.

### Transaction and Delivery Semantics

Domain update и outbox insert выполняются в одной database transaction.

HTTP response означает:

```text
business state зафиксирован
+
audit event надёжно записан в outbox
```

HTTP response не означает, что Kafka consumer уже получил сообщение.

Platform delivery имеет семантику at-least-once. Возможна повторная публикация одного `eventId`. Consumers обязаны выполнять deduplication по `eventId`.

### Failure Handling

| Ошибка | Поведение приложения | Ответственность |
|---|---|---|
| Не удалось записать outbox record | Transaction rollback, API error | Application + platform library |
| Kafka недоступна после commit | Отзыв остаётся успешным, platform retry | Platform |
| Schema validation failed | Outbox record не публикуется, alert | Application config + platform |
| Retry exhausted | Перемещение в DLQ и alert | Platform |
| Неверный topic mapping | Deployment/configuration error | Application |

Rejected business operations не создают `APPROVAL_REQUEST_WITHDRAWN`. Их security/technical logging выполняется существующими platform mechanisms.

## Observability

Проверяются platform metrics:

- audit outbox pending count;
- audit publish success rate;
- audit publish failure rate;
- oldest pending event age;
- DLQ count.

Application добавляет structured log без чувствительных данных:

```text
request_withdrawal_completed requestId=<id> actorId=<id> correlationId=<id>
```

Alert создаётся по platform standard для роста oldest pending event age и DLQ count.

## Data Impact

Новых business tables не требуется.

Используются:

- существующие request status и version;
- существующий approval started marker;
- platform-managed outbox table.

Нужна проверка, что outbox migration уже установлена целевой версией platform runtime.

## UI Changes

Action отображается, если:

- status равен `Submitted`;
- approval started flag равен `false`;
- пользователь является author;
- effective permissions содержат `approval-request.withdraw-own`.

Перед отправкой UI показывает confirmation dialog. При `409` UI обновляет заявку и показывает message, соответствующий `error.code`.

## Backward Compatibility

- Существующие endpoints и response models не изменяются.
- Добавляется новый endpoint.
- Новый permission выдаётся существующим ролям через configuration.
- Kafka event использует новую event type и schema version `1`.
- Consumers, не подписанные на event type, не затрагиваются.

## Rollout

1. Проверить наличие platform outbox migration.
2. Зарегистрировать topic ACL для application service account средствами платформы.
3. Deploy конфигурация приложения с выключенным feature flag.
4. Deploy backend.
5. Выполнить smoke test на TEST.
6. Включить backend feature flag для внутренней группы.
7. Проверить API, outbox и Kafka delivery metrics.
8. Deploy UI action.
9. Постепенно включить feature для всех пользователей.

## Rollback

- Выключить feature flag и скрыть UI action.
- Не удалять уже созданные audit events.
- Новый endpoint возвращает standard feature-disabled response.
- Database rollback не требуется.
- Topic mapping сохраняется для доставки pending outbox records.

## Test Strategy

### Unit Tests

- ownership rule;
- allowed status;
- approval started;
- transition result;
- audit event payload mapping;
- sensitive fields отсутствуют.

### Integration Tests

- status update и outbox insert атомарны;
- rollback при ошибке outbox insert;
- optimistic concurrency со стартом согласования;
- конфигурация приложения разрешается в ожидаемый topic alias.

### API Tests

- `200`, `403`, `404`, `409`, `412`;
- response schema;
- permission enforcement;
- повторный запрос;
- correlation ID propagation.

### Contract Tests

- audit JSON соответствует schema version `1`;
- обязательные fields присутствуют;
- неизвестные запрещённые fields отсутствуют;
- `eventId` стабилен при retry одной outbox record.

### Manual QA

- UI visibility для author/non-author;
- успешный отзыв;
- race со стартом согласования;
- сообщение для каждого `error.code`;
- audit event найден по correlation ID;
- Kafka outage не откатывает уже committed withdrawal.

### Regression

- отправка заявки;
- старт согласования;
- approve/reject flow;
- внутренний инструмент поддержки;
- существующие audit events;
- role mappings других операций.

## Risks and Mitigations

| Риск | Mitigation |
|---|---|
| Прямое изменение статуса обходит rule | ADR-001 и PR search по status update |
| Race со стартом согласования | Atomic transition и concurrency test |
| Duplicate Kafka delivery | `eventId` и consumer deduplication |
| Неверный PROD topic | Configuration validation до deploy |
| PII в audit payload | Allowlist attributes и contract test |
| Platform outbox backlog | Metrics, alert и DLQ procedure |
| UI показывает устаревший action | Backend remains authoritative |

## ADR Decision

Создаётся ADR-001 для общего правила переходов статуса через `RequestStatusTransitionService`.

Отдельный ADR для Kafka audit не создаётся: change использует существующий корпоративный platform standard и не принимает новое архитектурное решение о механизме доставки.

## Design Harvest Notes

После реализации проверить:

- соответствует ли фактический audit payload design;
- не появились ли дополнительные error codes;
- подтверждена ли atomicity outbox;
- нужно ли обновить platform integration documentation;
- добавлен ли change в ADR traceability index;
- отражает ли master spec только наблюдаемое поведение, а не детали Kafka implementation.
