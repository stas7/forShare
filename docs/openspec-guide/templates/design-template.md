# Design: <change-name>

## Как использовать шаблон

Design описывает, как будет реализовано поведение из delta spec.

Не копируйте requirements в design без необходимости. Ссылайтесь на них и описывайте технический способ реализации.

Если раздел неприменим, укажите:

```text
Не применимо: <краткая причина>
```

Не удаляйте важный раздел молча: reviewer должен понимать, что вопрос рассмотрен.

## Summary

Кратко опишите технический подход:

- что меняется;
- где находится основная business logic;
- какие возможности платформы используются;
- есть ли migration, rollout или compatibility impact.

## Context and Current Architecture

Как затронутая часть системы работает сейчас?

Опишите только необходимый контекст:

- текущий request/data flow;
- основные components;
- существующие integrations;
- current конфигурация приложения;
- известные legacy constraints;
- подтверждённые проблемы current design.

Не объявляйте случайную деталь реализации requirement без подтверждения.

## Goals

Какие технические результаты должен обеспечить design?

- `<technical-goal>`
- `<technical-goal>`

## Non-Goals

Какие технические изменения намеренно не выполняются?

- `<non-goal>`
- `<non-goal>`

## Responsibility Boundary

Разделите ответственность явно.

### Application Business Logic

Что реализует приложение:

- domain rules;
- state transitions;
- validation;
- error mapping;
- формирование business events;
- наблюдаемое поведение.

### Application-Owned Configuration

Что приложение конфигурирует:

- permissions и role mappings;
- feature flags;
- event types;
- topic aliases и environment variables;
- timeouts и допустимые policy profiles;
- schema versions;
- allowlists attributes.

Укажите владельца configuration и способ validation.

### Platform Capabilities

Что предоставляет корпоративная платформа:

- authentication / authorization mechanism;
- audit transport;
- logging;
- message broker client;
- outbox;
- retry / DLQ;
- metrics / tracing;
- standard error handling.

Не описывайте возможность платформы как новую реализацию приложения.

## Proposed Solution

Опишите выбранное решение по шагам.

Для каждого существенного шага укажите:

- component owner;
- input;
- output;
- business или technical validation;
- side effects;
- поведение при ошибке.

## Request / Event / Data Flow

Покажите основной flow.

Пример:

```text
Client
→ API
→ platform access check
→ application handler
→ domain service
→ transaction
   ├── state change
   └── outbox record
→ response

Асинхронно:
outbox worker
→ platform producer
→ Kafka topic
→ consumer
```

Добавьте альтернативные flows для ошибок, retry или asynchronous processing, если они важны.

## Domain Rules and Invariants

Какие invariants обязаны сохраняться независимо от entry point?

- `<invariant>`
- `<invariant>`

Укажите, где они проверяются и почему выбран этот layer.

## Concurrency and Transaction Boundaries

Если применимо, опишите:

- возможные race conditions;
- optimistic или pessimistic locking;
- idempotency;
- transaction boundaries;
- atomic operations;
- порядок side effects;
- поведение duplicate request;
- consistency model.

## API Impact

Если API меняется, опишите:

- method и path;
- authentication и permission;
- request contract;
- success response;
- error codes;
- versioning;
- idempotency;
- backward compatibility.

Пример таблицы:

| HTTP | error.code | Условие |
|---|---|---|
| `<code>` | `<error-code>` | `<condition>` |

## Data Impact

Опишите:

- новые или изменённые entities/fields;
- database migrations;
- backfill;
- indexes и constraints;
- data retention;
- compatibility старых данных;
- rollback migration;
- ownership данных.

## Security and Access Control

Опишите:

- permission codes;
- role mappings;
- ownership checks;
- backend authorization boundary;
- защита от раскрытия существования чужого resource;
- чувствительные данные;
- secrets;
- audit requirements;
- abuse cases.

UI visibility не считается достаточной authorization.

## Platform Integration

Создайте отдельный subsection для каждой возможность платформы или внешней integration.

### Integration: <name>

#### Purpose

Зачем integration нужна change?

#### Application Responsibility

Какие business decisions, configuration и payload принадлежат приложению?

#### Platform / Provider Responsibility

Какие transport, retry, storage или operational capabilities предоставляет платформа?

#### Configuration

Где задаются:

- endpoint или topic alias;
- зависящий от environment values;
- permission / ACL;
- schema version;
- timeout / retry profile;
- feature flag.

Не храните credentials и secrets в design или repository configuration.

#### Contract

Опишите или приложите:

- request/message schema;
- required fields;
- identifiers;
- correlation ID;
- versioning;
- compatibility rules;
- запрещённые поля.

#### Delivery Semantics

Если применимо:

- synchronous / asynchronous;
- at-most-once / at-least-once / exactly-once expectations;
- ordering;
- deduplication;
- idempotency key;
- acknowledgement meaning;
- eventual consistency.

#### Failure Handling

| Failure | Application behavior | Platform behavior | Monitoring |
|---|---|---|---|
| `<failure>` | `<behavior>` | `<behavior>` | `<signal>` |

## Observability

Опишите:

- structured logs;
- metrics;
- traces;
- correlation IDs;
- dashboards;
- alerts;
- SLO/SLA impact;
- как команда подтвердит корректность после rollout.

Логи и events не должны содержать запрещённые чувствительные данные.

## Backward Compatibility

Что может сломаться для:

- существующих пользователей;
- API consumers;
- event consumers;
- старых application versions;
- integrations;
- environment configurations?

Как compatibility будет проверена?

## Migration Plan

Если применимо, укажите:

1. Подготовительные изменения.
2. Data/configuration migration.
3. Порядок deployments.
4. Dual-read / dual-write period.
5. Backfill.
6. Критерии завершения migration.

## Rollout Plan

Опишите:

- feature flag;
- environments;
- canary или internal group;
- порядок backend/configuration/UI deployment;
- smoke checks;
- monitoring period;
- критерии полного включения.

## Rollback Plan

Опишите:

- как остановить новое поведение;
- что происходит с уже записанными данными и events;
- можно ли откатить schema/configuration;
- как доставить pending messages;
- какие manual operations потребуются;
- критерий принятия решения о rollback.

Фраза «откатить PR» недостаточна для changes с data или asynchronous side effects.

## Test Strategy

### Unit Tests

Какие domain rules и mappings проверяются изолированно?

### Integration Tests

Какие transaction boundaries, repositories и platform adapters проверяются вместе?

### Contract Tests

Какие API/message schemas и compatibility rules проверяются?

### Concurrency / Resilience Tests

Какие races, retries, duplicates, timeouts и outages проверяются?

### Manual QA

Какие Scenarios проверяются вручную и в какой environment?

### Regression

Какие существующие flows должны остаться без изменений?

## Risks and Mitigations

| Risk | Impact | Probability | Mitigation | Owner |
|---|---|---|---|---|
| `<risk>` | `<impact>` | `<probability>` | `<mitigation>` | `<owner>` |

## Alternatives Considered

Для каждой существенной альтернативы укажите:

- вариант;
- преимущества;
- недостатки;
- почему он не выбран.

## ADR Decision

Ответьте явно:

```text
ADR required: yes / no
```

Если `yes`:

- какое долговременное архитектурное решение появилось;
- где будет создан ADR;
- какой existing ADR оно дополняет или supersedes.

Если `no`:

- почему change не создаёт нового долговременного архитектурного решения;
- на какой existing standard или ADR опирается решение.

## Design Harvest Result

Этот раздел обновляется после implementation и до archive.

Зафиксируйте:

- совпала ли реализация с design;
- какие решения изменились;
- что перенесено в ADR;
- что обновлено в architecture/testing docs;
- что осталось только в archived design;
- какие links нужно обновить после archive;
- какие follow-up tasks созданы.

`design.md` целиком не переносится в ADR. В ADR извлекаются только долговременные архитектурные решения, их причины и последствия.
