# Роль: QA

## Главная ответственность

QA отвечает за testability, evidence и уверенность в том, что реализованное behavior соответствует specification и не ломает критичный regression.

Роль QA может выполнять один или несколько участников. Для change должен быть определён owner test plan и QA sign-off.

## За что отвечает QA

- review proposal и Scenarios;
- testability requirements;
- baseline verification;
- test data;
- manual test plan;
- regression scope;
- exploratory testing;
- QA evidence;
- QA sign-off;
- сохранение постоянных regression lessons.

## Explore

QA подключается до implementation.

Он помогает определить:

- как воспроизвести текущее поведение;
- какие environments/configurations нужны;
- какие roles и permissions требуются;
- какие данные подготовить;
- какие соседние flows затронуты;
- какие unknowns мешают проверке.

Для brownfield QA подтверждает baseline и сохраняемое поведение. Для bugfix QA фиксирует defect evidence.

## Proposal Review

QA проверяет:

- понятны ли Success Criteria;
- можно ли проверить scope отдельно;
- указаны ли affected users и systems;
- описаны ли error и integration risks;
- нет ли скрытого behavior в Out of Scope;
- достаточно ли информации для test planning.

## Specification Review

QA проверяет каждый Requirement и Scenario.

Хороший Scenario отвечает на вопросы:

- какое исходное состояние;
- какое действие выполняется;
- какой результат ожидается;
- что не должно измениться;
- какую ошибку видит пользователь или система;
- как получить фактический result.

QA выявляет:

- negative cases;
- edge cases;
- boundary values;
- permissions/ownership cases;
- duplicate actions;
- concurrency behavior;
- integration unavailable behavior;
- backward compatibility risks.

Неприменимые Scenarios не добавляются формально.

## Design Review

QA читает design, чтобы понять:

- затронутые components;
- transaction и integration boundaries;
- configuration changes;
- failure handling;
- observability;
- migration/rollout/rollback;
- какие tests нужны на каждом layer.

QA не выбирает architecture вместо Developer, но может блокировать design, который невозможно надёжно проверить или диагностировать.

## Application Configuration и Environments

Если change зависит от configuration, QA проверяет:

- role/permission mappings;
- feature flags;
- topic/endpoint aliases;
- TEST/PROD differences;
- defaults;
- schema validation;
- rollback configuration;
- доступность test environment.

Configuration является частью test scope, если влияет на behavior.

## Test Strategy

QA определяет сочетание:

- automated coverage;
- manual checks;
- exploratory testing;
- regression;
- production verification.

QA согласует с Developer, какие tests находятся на unit, integration, API, contract и end-to-end levels.

## Test Data

QA отвечает за понятный plan test data:

- valid cases;
- boundary values;
- different roles;
- different states;
- failure responses;
- duplicate/concurrent cases;
- data cleanup;
- отсутствие production secrets и персональных данных.

## Test Expectation Conflicts

QA не должен автоматически считать existing test устаревшим.

При конфликте QA:

1. Фиксирует failing test.
2. Сравнивает actual и ожидаемое поведение.
3. Проверяет строгость assertions, fixtures и mocks.
4. Сверяет master spec и active change.
5. Участвует в определении defect code/requirement/test.
6. Ждёт явного решения перед изменением expectation.

QA блокирует попытки скрыть conflict через skip, ослабление assertion, broad retry или исключение из coverage.

## Manual QA

QA выполняет Scenarios и дополнительные checks.

Evidence включает:

- environment/build;
- test data;
- role/configuration;
- фактический result;
- logs/correlation IDs;
- screenshots или payloads, если уместно;
- найденные defects;
- итоговый status.

## Regression и Exploratory Testing

Regression scope определяется по:

- master spec;
- affected components;
- design boundaries;
- фактическому PR diff;
- historical defects;
- platform/configuration impact.

Exploratory testing проверяет неочевидные последовательности, races, stale state и environment differences.

## Platform Integrations

Если change использует возможность платформы, QA проверяет observable contract и application-owned configuration.

Примеры:

- permission действительно выдаётся нужной role;
- audit event имеет правильный type/payload;
- message попадает в configured topic;
- outage behavior соответствует spec;
- retry не создаёт недопустимый duplicate business effect;
- metrics/logs позволяют найти проблему.

QA не обязан повторно тестировать внутреннюю реализацию платформы вне contract change.

## QA Sign-Off

QA sign-off означает:

```text
Expected behavior подтверждён.
Critical regression пройден.
Known risks и defects зафиксированы.
Evidence относится к проверенной build/configuration.
```

Sign-off не выдаётся только потому, что automated tests зелёные.

## Design Harvest

Перед archive QA:

- фиксирует постоянные regression rules;
- проверяет соответствие Scenarios фактически проверенному behavior;
- определяет, нужны ли updates testing docs;
- добавляет lessons по test data/environments;
- участвует в review ADR, если decision создаёт постоянное test constraint.

## После Archive

QA проверяет:

- master Scenarios соответствуют QA result;
- важные regression cases не потерялись;
- archived change содержит QA-related context/tasks;
- post-archive validation прошла;
- follow-up defects имеют owners.

## Что QA не должен делать

QA не должен:

- подключаться только после merge;
- считать specification непроверяемой проблемой «аналитика»;
- менять ожидаемое поведение без Analyst;
- ослаблять tests ради green pipeline;
- тестировать только happy path;
- игнорировать configuration и platform boundary;
- выдавать sign-off без evidence;
- переносить все test details в master spec.

## Результат работы роли

После работы QA команда понимает:

- что и где проверено;
- какой regression выполнен;
- какие risks остались;
- соответствует ли implementation specification;
- можно ли безопасно выполнять rollout и archive.
