# PR: Add request withdrawal

## OpenSpec Change

`openspec/changes/archive/2026-07-06-add-request-withdrawal`

## Summary

Добавлена возможность автору отозвать собственную заявку до начала согласования.

Изменение включает backend authorization, атомарный status transition, UI action и business audit event через стандартный platform audit module и Kafka.

## Business Behavior

- Автор с permission `approval-request.withdraw-own` может отозвать заявку в статусе `Submitted`.
- После начала согласования отзыв отклоняется.
- Пользователь не может отозвать чужую заявку.
- Повторный отзыв не изменяет state и не создаёт duplicate business event.
- UI visibility не заменяет backend validation.

## Platform Boundary

Приложение реализует:

- ownership и withdrawal rules;
- status transition;
- permission и role mapping configuration;
- audit event type, payload и topic mapping;
- API и UI behavior.

Платформа предоставляет:

- authentication и permission enforcement;
- audit envelope;
- transactional outbox;
- Kafka producer;
- retry, DLQ, metrics и technical logs.

Новый Kafka producer в application code не добавлялся.

## API

Добавлен endpoint:

```text
POST /requests/{requestId}/withdraw
```

Ответы:

| HTTP | error.code |
|---|---|
| 200 | — |
| 403 | `REQUEST_WITHDRAWAL_FORBIDDEN` |
| 404 | `REQUEST_NOT_FOUND` |
| 409 | `APPROVAL_ALREADY_STARTED` |
| 409 | `REQUEST_STATUS_NOT_WITHDRAWABLE` |
| 409 | `REQUEST_ALREADY_WITHDRAWN` |
| 412 | `REQUEST_VERSION_MISMATCH` |

## Audit and Kafka

- Event type: `APPROVAL_REQUEST_WITHDRAWN`.
- Schema: `contracts/audit/approval-request-withdrawn-v1.schema.json`.
- Topic alias: `platform-audit`.
- Topic name: `${PLATFORM_AUDIT_TOPIC}`.
- Delivery: transactional outbox + at-least-once Kafka publishing.
- Deduplication key: `eventId`.
- State change и outbox insert выполняются в одной transaction.
- Kafka outage после commit не откатывает withdrawal; platform retry доставляет pending event.

Audit payload содержит request ID, actor ID, status transition, version и correlation ID. Содержимое заявки, tokens и PII не публикуются.

## Configuration

Изменён `config/application.yaml`:

- permission registration;
- role mappings;
- audit event registration;
- topic alias;
- schema version;
- allowed attributes.

TEST и PROD topic ACL подтверждены platform team.

## Architecture

- Withdrawal rule реализовано в `RequestWithdrawalService`.
- Все изменения статуса проходят через `RequestStatusTransitionService`.
- Race со стартом согласования разрешается внутри atomic transition.
- ADR-001 фиксирует общее правило status transitions.
- Отдельный ADR для platform audit не нужен: используется существующий corporate standard.

## Automated Testing

Этот раздел моделирует evidence из реального application repository. Production code и test suite не входят в documentation example, поэтому указанные tests нельзя запустить из текущей директории.

В реальном PR вместо примерных чисел указываются фактические команды, CI links и результаты конкретного commit.

Добавлены:

- 11 unit tests;
- 6 integration tests;
- 9 API tests;
- 4 audit contract tests;
- concurrency test withdrawal vs approval start.

Проверено:

```text
OpenSpec validation: passed
Static analysis: passed
Automated tests: 236 passed, 0 failed
Audit schema validation: passed
```

Числа приведены как часть реалистичного примера PR и должны заменяться фактическими результатами проекта.

## Manual QA

QA выполнено на TEST environment:

- успешный отзыв;
- пользователь без permission;
- отзыв чужой заявки;
- approval already started;
- repeated request;
- stale UI state;
- event найден в Kafka по correlation ID;
- Kafka outage и retry delivery;
- regression submission/approval flow.

QA sign-off: получен.

## Observability

После TEST rollout проверены:

- audit outbox pending count;
- publish success/failure rate;
- oldest pending event age;
- DLQ count;
- application logs по request ID и correlation ID.

Новых alerts не потребовалось; используются platform standard thresholds.

## Rollout

- Backend deployment выполняется с выключенным feature flag.
- После smoke test feature включается для внутренней группы.
- UI deploy выполняется после проверки backend и audit delivery.
- Полное включение выполняется после 24 часов наблюдения.

## Rollback

- Выключить feature flag.
- Скрыть UI action.
- Сохранить topic mapping до доставки pending outbox records.
- Database rollback не требуется.

## Risks

| Риск | Состояние |
|---|---|
| Legacy direct status updates | Не входят в scope; ADR-001 запрещает новые случаи |
| Duplicate Kafka delivery | Consumers используют `eventId` |
| Неверный topic configuration | Добавлена config validation |
| PII leakage | Allowlist + JSON schema contract test |
| Outbox backlog | Platform metrics и alerting |

## Archive Result

- Production rollout завершён.
- Production verification завершена.
- Delta spec применена к openspec/specs/approval-requests/spec.md.
- Change архивирован как 2026-07-06-add-request-withdrawal.
