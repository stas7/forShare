# Роль: Developer

## Главная ответственность

Developer отвечает за техническое решение, implementation и техническую целостность OpenSpec change.

Роль Developer может быть распределена между несколькими участниками, но для каждого change должен быть один понятный owner технического результата.

## За что отвечает Developer

- анализ technical impact;
- `design.md`;
- `tasks.md`;
- конфигурация приложения и contracts;
- production-код;
- automated tests;
- migration, rollout и rollback;
- PR description;
- Design Harvest;
- техническая часть archive и post-archive verification.

## До Implementation

Developer читает:

- project context;
- затронутую master spec;
- proposal;
- delta spec;
- existing ADR;
- relevant code, configuration и contracts.

Он не начинает code, пока существенные unknowns по behavior или architecture не закрыты.

## Proposal Review

Developer проверяет:

- техническую реализуемость What Changes;
- затронутые components и integrations;
- dependencies;
- migration impact;
- platform constraints;
- security и data risks;
- реалистичность scope.

Technical details добавляются в affected areas/risks или design, но не заменяют business Why.

## Design

Developer является owner `design.md`.

Design должен объяснять:

- current architecture;
- proposed solution;
- responsibility boundary;
- request/data/event flow;
- domain invariants;
- concurrency и transaction boundaries;
- API/data/security impact;
- integration contracts;
- observability;
- migration;
- rollout и rollback;
- test strategy;
- alternatives;
- ADR decision.

Если раздел неприменим, Developer указывает краткую причину.

## Application и Platform Boundary

Developer явно разделяет:

```text
Application business logic
Application-owned configuration
Platform capabilities
```

Приложение не должно повторно реализовывать стандартный platform mechanism.

При этом бизнес-правила не должны исчезать внутри platform configuration. Permission codes, event types, topic mappings и payload contracts остаются ответственностью приложения, если они определяются change.

## Tasks

Developer является owner `tasks.md`.

Tasks должны:

- соответствовать proposal/spec/design;
- быть проверяемыми;
- включать testing и review;
- отражать configuration/contracts/migration, если применимо;
- включать Design Harvest и archive;
- обновляться по фактическому состоянию.

Project-specific детализация tasks определяется командой и репозиторием.

## Implementation

Во время implementation Developer:

- реализует behavior из delta spec;
- следует reviewed design;
- поддерживает artifacts актуальными;
- не включает несвязанный refactoring;
- сохраняет пользовательские изменения;
- не обходит errors silent fallback;
- не hardcode зависящий от environment configuration;
- не хранит secrets в repository;
- добавляет diagnostics без чувствительные данные.

Если implementation требует иного поведения, сначала обновляется delta spec. Если меняется технический подход, обновляется design.

## Contracts и Configuration

Developer отвечает за:

- API/message/schema contracts;
- versioning и compatibility;
- permission/configuration schemas;
- environment variable references;
- syntax/schema validation;
- migration и rollback configuration;
- отсутствие credentials и secrets.

Configuration, меняющая behavior, проходит полноценный review.

## Automated Tests

Developer добавляет tests в объёме, соответствующем риску:

- unit tests для domain rules;
- integration tests для boundaries;
- contract tests для API/messages;
- concurrency/resilience tests, если применимо;
- regression tests для сохраняемого поведения.

Developer не изменяет test expectation автоматически ради прохождения нового code.

При конфликте он показывает actual/expected, объясняет вероятную причину и запрашивает решение по behavior.

## PR

Developer является owner PR description.

PR должен содержать:

- ссылку на OpenSpec change;
- реализованные Requirements;
- существенные design decisions;
- configuration/contracts impact;
- фактические test commands и results;
- manual QA instructions;
- risks;
- rollout/rollback;
- remaining work.

Evidence должно относиться к текущему commit SHA.

## QA Support

Developer:

- предоставляет build/environment instructions;
- готовит diagnostics;
- помогает с test data;
- исправляет findings;
- обновляет artifacts, если findings меняют behavior или design;
- не объявляет QA result вместо QA.

## Design Harvest

Перед archive Developer:

- сравнивает design с фактической реализацией;
- обновляет design;
- извлекает долговременные архитектурное решениеs в ADR;
- обновляет architecture docs;
- фиксирует follow-up debt;
- подтверждает, что полный design останется в archived change;
- классифицирует change как architectural или non-architectural.

`design.md` целиком не копируется в ADR.

## Archive

Developer обычно выполняет техническую операцию archive:

```text
openspec validate <change-name> --strict
openspec archive <change-name> --yes
openspec validate --all --strict
```

После archive Developer проверяет:

- master spec и `Purpose`;
- dated archive directory;
- полноту archived artifacts;
- ADR source links;
- traceability index;
- Git diff.

OpenSpec CLI не выполняет QA, Design Harvest, ADR, commit или push автоматически.

## Что Developer не должен делать

Developer не должен:

- позволять code молча определять requirement;
- менять master spec напрямую для future behavior;
- переносить детали реализации в specification;
- ослаблять tests ради нового code;
- скрывать platform failure;
- смешивать несвязанный refactoring с change;
- создавать ADR для каждого технического шага без долговременного решения;
- считать archive завершённым без post-archive verification.

## Результат работы роли

После работы Developer:

- solution соответствует ожидаемое поведение;
- implementation соответствует design;
- configuration и contracts reviewable;
- tests подтверждают result;
- risks и operations понятны;
- architecture knowledge сохранено;
- archive корректно сформирован.
