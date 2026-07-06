# Pull Request: <short-title>

## Как использовать шаблон

Замените placeholders фактическими данными текущего PR.

Не указывайте примерные test results, вымышленные CI links или неподтверждённый QA sign-off.

Если раздел неприменим, укажите краткую причину:

```text
Не применимо: <почему>
```

## OpenSpec Change

```text
openspec/changes/<change-name>
```

Укажите ссылку на active change и затронутые capabilities.

## Why

Какую проблему решает PR и почему change нужен?

Кратко перескажите Why из proposal, не копируя весь документ.

## What Changed

Что фактически изменено в этом PR?

- `<изменение behavior>`
- `<изменение implementation>`
- `<изменение configuration/contract>`

Укажите Out of Scope, если по diff граница может быть непонятна.

## Implemented Requirements

Перечислите точные Requirements и ключевые Scenarios:

- `<Requirement name>`
  - `<Scenario name>`

Если Requirement реализован частично, PR не должен представлять его как завершённый.

## Design Notes

Опишите только существенные технические решения:

- выбранный approach;
- transaction/concurrency boundaries;
- compatibility;
- важные alternatives;
- отклонения от reviewed design.

Если implementation изменила approach, сначала обновите `design.md`.

## Application and Platform Boundary

Если применимо, разделите:

```text
Application business logic:
- <что реализует приложение>

Application-owned configuration:
- <permissions, mappings, topics, flags, schema versions>

Platform capabilities:
- <что предоставляет платформа>
```

Не описывайте platform mechanism как новую application implementation.

## Configuration Changes

Если применимо, перечислите:

- изменённые configuration files;
- permissions и role mappings;
- feature flags;
- topic/endpoint aliases;
- environment variables;
- defaults;
- validation;
- rollback configuration.

Подтвердите, что secrets и credentials не добавлены в repository.

## API / Message / Schema Contracts

Если применимо, укажите:

- contract path;
- version;
- added/changed/removed fields;
- error codes;
- compatibility impact;
- consumer impact;
- contract tests;
- migration/deprecation plan.

## Data and Migration

Если применимо, опишите:

- schema migration;
- backfill;
- indexes/constraints;
- data compatibility;
- rollout order;
- rollback limitations;
- validation result.

## Security and Access

Если применимо, опишите:

- authentication;
- permission;
- role mapping;
- ownership checks;
- sensitive data;
- audit;
- security review result.

UI visibility не заменяет backend authorization.

## Automated Testing

Укажите фактические команды и результаты для текущего commit SHA.

```text
Commit SHA: <sha>
OpenSpec validation: <command + result>
Static analysis: <command + result>
Unit tests: <command + result>
Integration tests: <command + result>
Contract tests: <command + result>
Other: <command + result>
```

CI links:

- `<link>`

Не скрывайте failing tests через skip, ослабление assertions, изменение fixtures или снижение coverage.

## Test Expectation Changes

```text
Existing test expectations changed: yes / no
```

Если `yes`, укажите:

- failing test;
- actual behavior;
- previous expected behavior;
- согласованное новое expected behavior;
- ссылку на обновлённую delta spec;
- кто подтвердил изменение.

## Manual QA

Укажите:

- environment/build;
- выполненные Scenarios;
- roles/configuration;
- test data;
- фактические results;
- defects;
- QA evidence links;
- QA sign-off status.

```text
QA sign-off: received / pending / not applicable
```

## Regression

Какие существующие flows проверены?

- `<regression flow>`

Объясните scope regression относительно design и фактического diff.

## Observability

Если применимо:

- logs;
- metrics;
- traces;
- correlation IDs;
- dashboards;
- alerts;
- smoke/production verification.

Подтвердите отсутствие чувствительных данных.

## Rollout Plan

Если применимо:

1. `<deployment/configuration step>`
2. `<feature flag/canary step>`
3. `<verification step>`
4. `<full rollout criterion>`

## Rollback Plan

Опишите:

- trigger;
- commands/actions;
- data/message consequences;
- configuration rollback;
- pending events;
- verification after rollback.

Фраза «откатить PR» недостаточна для changes с data, configuration или asynchronous side effects.

## ADR Decision

```text
ADR required: yes / no
```

Если `yes`:

- ADR: `<path/link>`
- Decision: `<краткое решение>`

Если `no`:

- Reason: `<почему новое долговременное архитектурное решение не появилось>`
- Existing ADR/standard: `<ссылка или не применимо>`

## Risks and Known Limitations

| Risk / limitation | Impact | Mitigation / owner |
|---|---|---|
| `<risk>` | `<impact>` | `<mitigation>` |

## Remaining Work

- `<task + owner>`

Если remaining work блокирует Requirement, PR не готов к merge.

## Review Checklist

```text
[ ] OpenSpec artifacts синхронизированы с implementation.
[ ] Фактические commands/results указаны.
[ ] Configuration/contracts прошли review.
[ ] Risks и rollback понятны.
[ ] QA status указан.
[ ] ADR decision указан.
[ ] Remaining work имеет owners.
```
