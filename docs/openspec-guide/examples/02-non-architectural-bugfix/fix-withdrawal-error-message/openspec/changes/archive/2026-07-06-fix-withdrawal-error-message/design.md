# Design: fix-withdrawal-error-message

## Summary

Добавляется отдельный mapping существующей domain error `ApprovalAlreadyStarted` в API error code `APPROVAL_ALREADY_STARTED` и UI message catalog.

Business rule и status transition не меняются.

## Current Behavior

Domain service уже различает причину `ApprovalAlreadyStarted`, но API mapper объединяет её с другими withdrawal conflicts:

```text
ApprovalAlreadyStarted
→ HTTP 409
→ REQUEST_STATUS_NOT_WITHDRAWABLE
```

UI использует общий fallback message.

## Proposed Solution

Изменить mapping:

```text
ApprovalAlreadyStarted
→ HTTP 409
→ APPROVAL_ALREADY_STARTED
→ withdrawal.approvalAlreadyStarted
```

Другие domain errors продолжают использовать существующие mappings.

## Responsibility Boundary

### Application Business Logic

Business rule не меняется. Приложение только сохраняет конкретную причину отказа при преобразовании domain error в API/UI contract.

### Application-Owned Configuration

В message catalog добавляется ключ:

```text
withdrawal.approvalAlreadyStarted
```

### Platform Capabilities

Используется существующий standard API error envelope и существующий localization mechanism. Изменения платформы не требуются.

## API Impact

| Условие | HTTP | До change | После change |
|---|---:|---|---|
| Approval started | 409 | `REQUEST_STATUS_NOT_WITHDRAWABLE` | `APPROVAL_ALREADY_STARTED` |
| Wrong status | 409 | `REQUEST_STATUS_NOT_WITHDRAWABLE` | без изменений |
| Already withdrawn | 409 | `REQUEST_ALREADY_WITHDRAWN` | без изменений |

## Backward Compatibility

Изменяется error code только для конкретной причины approval started. Известные consumers проверены: UI использует новый code, внешних consumers этого endpoint нет.

## Test Strategy

- Unit test error mapper.
- API test `409 / APPROVAL_ALREADY_STARTED`.
- UI test message mapping.
- Regression других withdrawal conflicts.
- Manual QA с заявкой, для которой согласование уже началось.

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Изменится code другого conflict | Parameterized mapper regression test |
| UI не знает новый code | UI mapping включён в тот же change |
| Message расходится между clients | Message catalog key review |

## ADR Decision

```text
ADR required: no
```

Change не создаёт нового долговременного архитектурного решения. Он использует существующие domain errors, API error envelope и localization pattern.

## Design Harvest Result

- Реализация соответствует design.
- Новых architecture constraints не появилось.
- ADR не создан.
- Change классифицирован как `Non-architectural`.
- Подробный mapping остаётся в archived design и API tests.
