# OpenSpec Team Guide

Практическое руководство для команды, которая впервые внедряет OpenSpec.

Руководство помогает договориться до начала кода:

- зачем нужен change;
- какое поведение изменяется;
- кто отвечает за artifacts и review;
- как описать техническое решение;
- как проверить implementation;
- что происходит при Design Harvest и archive;
- где сохраняются master spec, ADR и история change.

## С чего начать

Основная точка входа:

[docs/openspec-guide/00-start-here.md](docs/openspec-guide/00-start-here.md)

Рекомендуемый базовый маршрут:

1. [Общий workflow](docs/openspec-guide/01-workflow.md)
2. [Ownership artifacts](docs/openspec-guide/02-file-ownership.md)
3. [Состав команды](docs/openspec-guide/03-team-composition.md)
4. [Инструкции для AI-агентов](docs/openspec-guide/04-agent-instructions.md)
5. [Glossary и правила языка](docs/openspec-guide/05-glossary.md)
6. [Классификация change](docs/openspec-guide/06-change-classification.md)
7. [OpenSpec commands и роли](docs/openspec-guide/07-openspec-commands.md)

## Процессы

Выберите процесс по типу работы:

- [Новый проект](docs/openspec-guide/processes/new-project.md)
- [Существующая legacy-система](docs/openspec-guide/processes/brownfield.md)
- [Bugfix](docs/openspec-guide/processes/bugfix.md)

## План первой недели

- [Новая функциональность на корпоративной платформе](docs/openspec-guide/99-first-week-plan.md)
- [Доработка legacy-системы](docs/openspec-guide/99-first-week-legacy-plan.md)

## Роли

- [Analyst](docs/openspec-guide/roles/analyst.md)
- [Developer](docs/openspec-guide/roles/developer.md)
- [QA](docs/openspec-guide/roles/qa.md)
- [Reviewer](docs/openspec-guide/roles/reviewer.md)

Роли являются общими. Конкретное распределение между людьми определяется командой.

## Шаблоны

- [Proposal](docs/openspec-guide/templates/proposal-template.md)
- [Design](docs/openspec-guide/templates/design-template.md)
- [Tasks](docs/openspec-guide/templates/tasks-template.md)
- [Pull Request](docs/openspec-guide/templates/pull-request-template.md)
- [ADR](docs/openspec-guide/templates/adr-template.md)
- [ADR Traceability Index](docs/openspec-guide/templates/adr-index-template.md)
- [Корневой AGENTS.md](docs/openspec-guide/templates/agents-root-template.md)
- [OpenSpec AGENTS.md](docs/openspec-guide/templates/openspec-agents-template.md)

## Checklists

- [Spec Review](docs/openspec-guide/checklists/spec-review-checklist.md)
- [PR Review](docs/openspec-guide/checklists/pr-review-checklist.md)
- [Archive](docs/openspec-guide/checklists/archive-checklist.md)

Archive checklist отдельно показывает:

- что команда делает вручную;
- какие commands запускает;
- что OpenSpec CLI выполняет автоматически;
- что команда обязана проверить после archive.

## Примеры

### Architectural Change

[add-request-withdrawal](docs/openspec-guide/examples/01-simple-feature/add-request-withdrawal/README.md)

Подробный завершённый change с:

- permission и status rules;
- API contract;
- platform audit через Kafka;
- application configuration;
- transactional outbox;
- tests и QA evidence;
- Design Harvest;
- ADR;
- master spec и archived change.

### Non-Architectural Bugfix

[fix-withdrawal-error-message](docs/openspec-guide/examples/02-non-architectural-bugfix/fix-withdrawal-error-message/README.md)

Компактный завершённый bugfix, который показывает:

- изменение observable error contract;
- `ADR required: no`;
- Design Harvest без создания ADR;
- классификацию `Non-architectural`;
- archive и обновление master spec.

## Главная идея

OpenSpec нужен не для формального заполнения файлов.

Он нужен, чтобы сохранить три вида знания:

```text
Master spec
→ как система должна работать сейчас

ADR
→ почему принято долговременное архитектурное решение

Archived change
→ полная история конкретного изменения
```

`design.md` целиком не переносится в ADR. При Design Harvest из него извлекаются только долговременные архитектурные решения; полный технический контекст остаётся в archived change.
