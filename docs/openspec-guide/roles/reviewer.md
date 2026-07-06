# Роль: Reviewer

## Главная ответственность

Reviewer выполняет независимую проверку согласованности behavior, техническое решение, implementation и evidence.

Reviewer — это роль, а не обязательно отдельный человек на всех стадиях. Важно, чтобы автор существенного решения не был единственным, кто его проверяет.

## За что отвечает Reviewer

- независимый Spec Review;
- Architecture / Design Review;
- PR Review;
- проверка contracts и platform boundary;
- проверка test evidence;
- проверка ADR decision;
- review post-archive result.

Reviewer не становится Owner artifacts и не переписывает их вместо автора.

## Review Principles

Хороший review comment содержит:

- конкретное наблюдение;
- нарушенное requirement/constraint;
- риск;
- ожидаемый критерий исправления.

Плохо:

```text
Сделано неправильно.
```

Хорошо:

```text
Ownership проверяется только в UI, но Scenario требует отказ backend для non-author.
Добавьте backend validation и API test для 403.
```

## Proposal Review

Reviewer проверяет:

- Why и business value понятны;
- What Changes имеет ограниченный scope;
- Out of Scope не противоречит Success Criteria;
- affected areas и risks полны;
- change не объединяет независимые задачи;
- platform assumptions подтверждены.

## Spec Review

Reviewer проверяет:

- master spec прочитана;
- delta operations выбраны правильно;
- Requirements не противоречат другим capabilities;
- SHALL и Scenarios корректны;
- behavior observable и проверяемо;
- детали реализации не подменяют requirement;
- access/errors/edge cases учтены, если применимо;
- regression behavior не удалено случайно.

Reviewer может оставить blocking comment до начала code.

## Architecture / Design Review

Reviewer проверяет:

- responsibility boundaries;
- domain invariants;
- transaction/concurrency model;
- API/data/security impact;
- integration contracts;
- compatibility;
- observability;
- migration/rollout/rollback;
- test strategy;
- alternatives;
- ADR decision.

Review оценивает не только возможность реализовать solution, но и способность безопасно его поддерживать и диагностировать.

## Platform Boundary

Reviewer проверяет:

```text
Application business logic
Application-owned configuration
Platform capabilities
```

Он блокирует:

- повторную реализацию standard platform mechanism без причины;
- перенос бизнес-правило внутрь непрозрачной configuration;
- hardcoded topics/endpoints/environments;
- отсутствие failure semantics;
- contracts без versioning;
- secrets в repository.

## PR Review

Reviewer сверяет:

```text
proposal
→ delta spec
→ design
→ tasks
→ code
→ configuration/contracts
→ tests/evidence
```

Он проверяет фактический diff, а не только PR description.

Если implementation расходится с design/spec, Reviewer требует синхронного обновления artifacts, а не устного объяснения.

## Test Expectations

Reviewer защищает existing test expectations.

При conflict он требует:

- failing test;
- actual/ожидаемое поведение;
- связь с master spec/active change;
- явное решение о behavior;
- отсутствие скрытого ослабления assertions/fixtures/coverage.

Green pipeline не является достаточным evidence, если tests были ослаблены.

## Test Evidence

Reviewer проверяет:

- команды и CI links;
- соответствие evidence текущему SHA;
- negative/error coverage;
- contract/concurrency/resilience tests, если применимо;
- manual QA readiness;
- regression scope.

Reviewer не обязан повторять весь QA, но должен видеть достаточные доказательства.

## Security and Operability

Reviewer проверяет:

- backend authorization;
- чувствительные данные;
- error disclosure;
- migration safety;
- logs/metrics/traces;
- alerting;
- rollout/rollback;
- operational ownership.

## ADR

Reviewer проверяет решение `ADR required: yes/no`.

Если ADR нужен:

- decision долговременное;
- Context объясняет проблему;
- Decision сформулирован однозначно;
- Consequences честно описаны;
- alternatives зафиксированы;
- sources ссылаются на archived change/design;
- supersession rules соблюдены.

Если ADR не нужен, design должен ссылаться на existing standard/ADR или объяснять локальность решения.

## Design Harvest

Reviewer проверяет, что:

- design обновлён по факту;
- architecture knowledge не осталось только в PR comments;
- долговременные decisions вынесены в ADR;
- полный design остаётся в archive;
- behavior остаётся в spec, а не переносится в ADR;
- follow-up debt имеет owners.

## Archive Review

После archive Reviewer проверяет:

- master spec получила ожидаемую delta;
- `Purpose` заполнен;
- archived change полный;
- ADR source links указывают на archived paths;
- traceability index обновлён;
- `openspec validate --all --strict` прошла;
- Git diff соответствует ожидаемому archive result.

## Blocking и Non-Blocking Comments

Blocking comment используется, если:

- нарушен requirement;
- есть security/data risk;
- design создаёт недопустимый architecture debt;
- tests/evidence недостаточны для риска;
- отсутствует migration/rollback для destructive change;
- OpenSpec artifacts расходятся с implementation.

Non-blocking comment используется для improvement, которое не мешает безопасному merge. Follow-up должен иметь owner, если работа действительно нужна.

## Что Reviewer не должен делать

Reviewer не должен:

- переписывать artifact вместо Owner;
- добавлять новый scope через comment без обновления proposal;
- требовать любимый pattern без объяснения риска;
- одобрять conflict spec/code «поправим позже»;
- считать platform standard достаточным без проверки конфигурация приложения;
- давать approval без просмотра tests/evidence;
- блокировать по вкусу, не указывая constraint.

## Результат работы роли

После работы Reviewer команда получает независимое подтверждение, что:

- behavior согласовано;
- solution технически обосновано;
- implementation и contracts соответствуют design;
- evidence достаточно;
- architecture knowledge сохранено;
- change готов двигаться к следующему gate.
