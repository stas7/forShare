# Example: add-request-withdrawal

Сквозной пример реального OpenSpec change, прошедшего полный цикл до archive.

Задача:

> Автор заявки должен иметь возможность отозвать её до начала согласования. Успешный отзыв должен попасть в корпоративный audit через стандартный platform audit module и Kafka.

## Состояние примера

Пример показывает завершённый change:

```text
proposal
→ delta spec
→ design
→ tasks
→ implementation
→ PR Review
→ QA
→ production verification
→ Design Harvest
→ archive
```

После archive:

- актуальное поведение находится в `openspec/specs/approval-requests/spec.md`;
- история изменения находится в `openspec/changes/archive/2026-07-06-add-request-withdrawal/`;
- архитектурное решение находится в `docs/adr/ADR-001-request-status-transitions.md`;
- конфигурация приложения и audit schema остаются рядом с примером реализации.

## Граница учебного примера

Репозиторий содержит OpenSpec artifacts, конфигурация приложения, audit schema, PR description и ADR, но не содержит production исходный код приложения и его test suite.

Поэтому нужно различать два слоя:

```text
Что реально можно проверить в этом documentation repository:
- OpenSpec master spec и archive structure;
- `openspec validate --all --strict`;
- синтаксис конфигурация приложения;
- синтаксис и структуру audit JSON schema;
- согласованность proposal, spec, design, tasks, PR и ADR.

Что моделирует pull-request.md как evidence из реального application repository:
- unit, integration, API и contract tests;
- static analysis;
- manual QA;
- Kafka outage test;
- production rollout и verification.
```

Числа tests, environment names и результаты QA в PR являются реалистичным примером оформления evidence. При использовании шаблона команда заменяет их фактическими командами, ссылками и результатами своего проекта.

Отсутствие implementation code в учебном примере не означает, что automated tests необязательны для реального change.

## Что показано

- текущее поведение и бизнес-проблема;
- scope, non-goals и success criteria;
- permission и ownership rules;
- race между отзывом и стартом согласования;
- API contract и business errors;
- разделение application logic, конфигурация приложения и возможности платформы;
- transactional outbox и at-least-once Kafka delivery;
- audit payload и JSON schema;
- зависящий от environment topic configuration;
- observability, rollout и rollback;
- подробные automated и manual checks;
- PR description с evidence;
- Design Harvest;
- ADR и traceability index;
- master spec и archived change;
- защита существующих test expectations.

## Структура после archive

```text
add-request-withdrawal/
├── AGENTS.md
├── README.md
├── pull-request.md
├── config/
│   └── application.yaml
├── contracts/
│   └── audit/
│       └── approval-request-withdrawn-v1.schema.json
├── docs/
│   └── adr/
│       ├── README.md
│       └── ADR-001-request-status-transitions.md
└── openspec/
    ├── AGENTS.md
    ├── specs/
    │   └── approval-requests/
    │       └── spec.md
    └── changes/
        └── archive/
            └── 2026-07-06-add-request-withdrawal/
                ├── proposal.md
                ├── design.md
                ├── tasks.md
                └── specs/
                    └── approval-requests/
                        └── spec.md
```

## Как читать пример

Рекомендуемый порядок:

```text
archived proposal.md
→ archived delta spec
→ archived design.md
→ config/application.yaml
→ audit JSON schema
→ archived tasks.md
→ pull-request.md
→ master spec
→ ADR-001
```

Archived delta spec показывает, какое поведение добавлялось конкретным change.

Master spec показывает актуальное поведение после применения delta.

Archived design описывает технический способ: domain service, atomic transition, outbox, Kafka topic mapping, retry и metrics.

`config/application.yaml` показывает configuration, которой владеет приложение.

JSON schema показывает contract сообщения, передаваемого platform audit module в Kafka.

ADR сохраняет долговременное правило status transitions, но не копирует весь design.

## Что произошло при archive

Перед archive команда выполнила Design Harvest и распределила знания из change по трём местам:

```text
Поведение из delta spec
→ master spec

Долговременное архитектурное решение из design.md
→ ADR

Полный технический контекст, альтернативы, rollout и rollback
→ archived design.md
```

В этом примере из `design.md` в ADR-001 вынесено долговременное правило:

```text
Все изменения статуса заявки проходят через RequestStatusTransitionService.
```

ADR фиксирует само решение, причины и последствия. Он не копирует `design.md` целиком. Подробности реализации отзыва, concurrency, transactional outbox, Kafka delivery, rollout и rollback остаются в archived `design.md`.

Отдельный ADR для platform audit не создан, потому что приложение использует уже принятое корпоративное архитектурное решение. Application-specific topic mapping, event payload и schema остаются в archived design, configuration и contract files.

После Design Harvest OpenSpec выполнил две операции:

1. Применил три Requirements из delta spec к master spec `approval-requests`.
2. Переместил change в archive с именем `2026-07-06-add-request-withdrawal`.

Таким образом:

- master spec содержит актуальное наблюдаемое поведение;
- ADR-001 содержит извлечённое из design долговременное архитектурное решение;
- archived change хранит полный proposal, delta spec, design и tasks;
- configuration и audit schema сохраняют application-specific platform contract.

Перед archive все tasks были закрыты, QA sign-off и production verification получены, а `Purpose` master spec был заменён с автоматически созданного `TBD` на содержательное описание capability.

## AGENTS.md

Корневой `AGENTS.md` задаёт общие обязательные правила. `openspec/AGENTS.md` уточняет работу с OpenSpec artifacts, но не ослабляет общие ограничения.
