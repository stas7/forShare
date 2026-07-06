# Checklist: PR Review

PR Review проверяет не только code diff, но всю цепочку:

```text
proposal
→ delta spec
→ design
→ tasks
→ implementation
→ configuration / contracts
→ tests / evidence
```

Пункты с пометкой «если применимо» не требуют искусственного artifact. Reviewer должен убедиться, что вопрос рассмотрен и действительно неприменим.

## 1. Scope и OpenSpec

```text
[ ] PR ссылается на active OpenSpec change.
[ ] PR соответствует Why и What Changes из proposal.
[ ] Out of Scope не попал в implementation незаметно.
[ ] Нет несвязанного refactoring или configuration changes.
[ ] Delta spec отражает финальное наблюдаемое поведение PR.
[ ] Code реализует все затронутые Scenarios.
[ ] design.md соответствует фактическому technical approach.
[ ] tasks.md отражает реальное состояние работы.
[ ] Изменение behavior сначала отражено в delta spec.
[ ] Изменение technical approach отражено в design.md.
[ ] Все review comments, меняющие решение, синхронизированы с artifacts.
```

## 2. Implementation

```text
[ ] Business rules находятся в выбранном design layer.
[ ] Backend validation не заменена UI visibility.
[ ] Error behavior соответствует spec и API/message contracts.
[ ] State transitions выполняются через согласованный mechanism.
[ ] Transaction boundaries соответствуют design.
[ ] Idempotency и duplicate requests обработаны, если применимо.
[ ] Race conditions рассмотрены, если применимо.
[ ] Нет новых stubs, silent fallback или broad exception handling, скрывающих ошибку.
[ ] Не изменены несвязанные участки или пользовательские изменения.
[ ] Temporary workaround имеет явный owner и follow-up, если применимо.
```

## 3. Application Configuration

Если PR меняет configuration:

```text
[ ] Configuration входит в scope proposal и описана в design.
[ ] Permission codes и role mappings согласованы.
[ ] Topic, endpoint и feature flag не hardcoded в production-код.
[ ] Environment-specific values задаются правильным способом.
[ ] Defaults безопасны и явно описаны.
[ ] Configuration проходит syntax/schema validation.
[ ] TEST и PROD differences рассмотрены.
[ ] Configuration rollback описан.
[ ] Credentials, tokens и secrets не попали в repository.
```

Configuration, меняющая behavior, проходит такой же review, как production-код.

## 4. API, Message и Schema Contracts

Если меняются contracts:

```text
[ ] Contract соответствует Requirements и design.
[ ] Required/optional fields определены.
[ ] Error codes и status codes определены.
[ ] Schema version и compatibility policy определены.
[ ] Consumer impact оценён.
[ ] Unknown/additional fields policy определена.
[ ] Correlation ID и business identifiers передаются корректно.
[ ] Idempotency/deduplication key определён, если применимо.
[ ] Contract tests добавлены.
[ ] Migration или deprecation plan описан, если применимо.
```

## 5. Platform Integrations

Если используются возможности платформы:

```text
[ ] Граница Application / Platform соответствует design.
[ ] Application не реализует повторно стандартный platform mechanism.
[ ] Application-owned бизнес-правила не спрятаны внутри platform details.
[ ] Application-owned configuration находится под review.
[ ] Delivery semantics понятны.
[ ] Retry, timeout, DLQ и поведение при сбое учтены, если применимо.
[ ] Platform outage behavior соответствует spec.
[ ] Operational ownership ошибки определён.
```

## 6. Security and Data

```text
[ ] Authentication и authorization выполняются на backend.
[ ] Ownership checks реализованы, если применимо.
[ ] Ответ не раскрывает чужой resource или чувствительные данные.
[ ] Input validation соответствует contract.
[ ] Logs, audit events и messages не содержат запрещённые данные.
[ ] Data retention и deletion requirements учтены, если применимо.
[ ] Database migration безопасна и проверена, если применимо.
[ ] Backfill и rollback данных описаны, если применимо.
[ ] Новые dependencies и permissions имеют минимально необходимый scope.
```

## 7. Test Expectations

```text
[ ] Существующие test expectations не удалены и не ослаблены без явного согласования.
[ ] При конфликте test и production-код показаны actual и ожидаемое поведение.
[ ] Решение о новом ожидаемое поведение подтверждено Analyst и отражено в delta spec.
[ ] Изменения assertions сохраняют необходимую строгость.
[ ] Изменения fixtures, mocks и golden files не скрывают изменение behavior.
[ ] Tests не отключены и не исключены из coverage ради прохождения PR.
[ ] Coverage threshold не снижен без отдельного решения.
[ ] Retry или exception handling не скрывает failing behavior.
```

## 8. Test Coverage and Evidence

```text
[ ] Unit tests покрывают существенные бизнес-правила.
[ ] Integration tests покрывают transaction/integration boundaries, если применимо.
[ ] Contract tests покрывают API/message schemas, если применимо.
[ ] Negative и error cases покрыты.
[ ] Permission и ownership cases покрыты, если применимо.
[ ] Concurrency, duplicate и retry cases покрыты, если применимо.
[ ] Regression-critical behavior покрыто.
[ ] Manual QA plan соответствует Scenarios.
[ ] Test evidence относится к текущему commit SHA.
[ ] В PR указаны фактические команды и результаты, а не примерные значения.
[ ] CI links доступны reviewer.
```

Reviewer запускает или проверяет результаты специфичный для проекта команд из корневого `AGENTS.md`.

Для active change обязательно выполняется:

```text
openspec validate <change-name> --strict
```

Warnings должны быть разобраны, даже если command завершилась с exit code `0`.

## 9. Observability and Operations

Если change влияет на runtime behavior:

```text
[ ] Structured logs достаточны для диагностики.
[ ] Correlation ID проходит через затронутый flow.
[ ] Metrics отражают success, failure и backlog, если применимо.
[ ] Alerts и dashboards обновлены или явно не требуются.
[ ] Logs/metrics не содержат чувствительные данные.
[ ] Smoke checks после deployment определены.
[ ] Production verification plan описан.
[ ] Operational owner известен.
```

## 10. Rollout and Rollback

Если change требует управляемого выпуска:

```text
[ ] Deployment order определён.
[ ] Feature flag или canary strategy описаны, если применимо.
[ ] Compatibility разных versions проверена.
[ ] Migration sequence проверена.
[ ] Rollback учитывает уже записанные данные и опубликованные events.
[ ] Pending messages не будут потеряны при rollback.
[ ] Критерии остановки rollout определены.
[ ] Критерии полного включения определены.
```

Фраза «откатить PR» не является достаточным rollback plan для changes с data, configuration или asynchronous side effects.

## 11. Architecture and Design Harvest Readiness

```text
[ ] Architecture constraints соблюдены.
[ ] Рассмотренные alternatives и выбранное решение актуальны.
[ ] В design.md есть явное `ADR required: yes / no` с обоснованием.
[ ] Новый долговременный архитектурное решение подготовлен для ADR, если применимо.
[ ] Existing ADR не противоречит implementation.
[ ] Решения, которые нужно сохранить при Design Harvest, обозначены.
[ ] Follow-up technical debt записан и не скрыт в PR comments.
```

PR Review не заменяет Design Harvest. На этом этапе reviewer только подтверждает, что необходимые знания не потеряны и могут быть извлечены перед archive.

## 12. Review Result

```text
[ ] Blocking comments закрыты.
[ ] Non-blocking follow-ups имеют owner.
[ ] Analyst review получен для изменений бизнес-поведение.
[ ] QA подтвердил готовность build и test instructions.
[ ] Technical / Architecture Reviewer дал approval.
[ ] PR готов к merge или явно возвращён на доработку.
```
