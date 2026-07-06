# ADR-001: Request status transitions must use transition service

## Status

Accepted

## Context

Статус заявки изменяется из нескольких мест: public API, internal workflow engine и support operations.

Если каждый component изменяет status напрямую:

- бизнес-правила применяются несогласованно;
- race conditions разрешаются по-разному;
- audit events могут не создаваться;
- новые status transitions сложно проверять централизованно.

Change `add-request-withdrawal` показал конкретный race между отзывом и стартом согласования.

## Decision

Все изменения статуса заявки SHALL проходить через `RequestStatusTransitionService`.

Controllers, jobs и workflow handlers не должны изменять status напрямую.

Transition service отвечает за:

- проверку разрешённого перехода;
- domain invariants;
- optimistic concurrency;
- единый result/error contract;
- создание связанных domain events в границах transaction.

## Consequences

Положительные последствия:

- status transition rules централизованы;
- API, workflow и support tools используют одинаковые правила;
- concurrency проверяется в одном месте;
- PR reviewers могут находить прямые status updates;
- automated tests покрывают transition matrix.

Отрицательные последствия:

- новые операции должны сначала расширить transition service;
- legacy direct updates нужно мигрировать отдельными changes;
- transition service становится критичным domain component.

## Alternatives Considered

### Проверять rule только в controller

Отклонено: internal jobs и support tools смогут обойти проверку.

### Проверять rule только в UI

Отклонено: UI не является security или consistency boundary.

### Разрешить каждому component собственную проверку

Отклонено: правила будут дублироваться и расходиться.

## Источники

- Archived change: `openspec/changes/archive/2026-07-06-add-request-withdrawal/`
- Design: `openspec/changes/archive/2026-07-06-add-request-withdrawal/design.md`
- Supersedes: не применимо
- Superseded by: не применимо
