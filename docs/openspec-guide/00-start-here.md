# 00. С чего начать

Этот документ для команды, которая впервые пробует OpenSpec.

## Простая идея

Без OpenSpec часто получается так:

```text
Задача → Код → PR → Споры → Тестирование → Через полгода никто не помнит, почему так сделали
```

С OpenSpec мы хотим работать так:

```text
Задача → Обсуждение → Спецификация → Design → Код → PR → QA → Archive → Актуальная спецификация
```

## Что такое OpenSpec в нашей работе

OpenSpec — это способ вести изменения через набор файлов:

```text
openspec/
├── specs/       # актуальное поведение системы
└── changes/     # активные изменения
```

Для каждой значимой задачи создаётся отдельный change:

```text
openspec/changes/<change-name>/
├── proposal.md  # зачем делаем и что входит в scope
├── design.md    # как собираемся реализовывать
├── tasks.md     # список работ
└── specs/       # изменения требований
```

## Главное правило

Если меняется поведение системы — это должно быть описано в OpenSpec.

Если изменяется только внутренний код без изменения поведения — OpenSpec может быть не нужен.

## Что не нужно делать на старте

Не надо пытаться сразу описать всю систему.

Начинаем с той части, которую реально меняем сейчас.

## Базовый маршрут чтения

Если команда впервые использует OpenSpec, начните с документов в таком порядке:

1. [01-workflow.md](01-workflow.md) — полный lifecycle change.
2. [02-file-ownership.md](02-file-ownership.md) — кто владеет каждым artifact.
3. [03-team-composition.md](03-team-composition.md) — роли команды из пяти человек.
4. [04-agent-instructions.md](04-agent-instructions.md) — правила для AI-агентов, test expectations и ADR traceability.
5. [05-glossary.md](05-glossary.md) — единые термины и правила языка.
6. [06-change-classification.md](06-change-classification.md) — когда нужны OpenSpec change, design, ADR и emergency exception.
7. [07-openspec-commands.md](07-openspec-commands.md) — кто, когда и какие OpenSpec commands запускает.

После этого выберите процесс под конкретную ситуацию.

## Какой процесс выбрать

| Ситуация | Документ | Что даёт |
|---|---|---|
| Новый проект без master spec | [processes/new-project.md](processes/new-project.md) | Первый небольшой vertical slice и первый полный OpenSpec cycle |
| Существующая legacy-система без полной спецификации | [processes/brownfield.md](processes/brownfield.md) | Исследование текущее поведение и постепенное создание master spec |
| Исправление дефекта | [processes/bugfix.md](processes/bugfix.md) | Expected behavior, root cause, regression и bugfix change |

Если задача совмещает несколько ситуаций, выберите основной процесс и добавьте необходимые проверки из другого.

Пример:

```text
Bugfix в legacy-системе
→ основной процесс bugfix
→ исследование текущее поведение и baseline из brownfield
```

## План внедрения на первую неделю

Для первого практического change используйте один из планов:

- [99-first-week-plan.md](99-first-week-plan.md) — новая функциональность на стандартизированной корпоративной платформе;
- [99-first-week-legacy-plan.md](99-first-week-legacy-plan.md) — доработка существующей legacy-системы.

План первой недели не заменяет основной process. Он помогает команде выбрать реалистичный scope и начать использовать процесс на практике.

## Роли

Подробные обязанности участников:

- [roles/analyst.md](roles/analyst.md) — Analyst / Expected Behavior Owner;
- [roles/developer.md](roles/developer.md) — Developer / Change Owner;
- [roles/qa.md](roles/qa.md) — QA Owner;
- [roles/reviewer.md](roles/reviewer.md) — Technical / Architecture Reviewer.

Если в команде два разработчика и два тестировщика, разделение ответственности описано в [03-team-composition.md](03-team-composition.md).

## Шаблоны

Шаблоны используются как стартовая структура, а не как формальная анкета:

- [templates/proposal-template.md](templates/proposal-template.md);
- [templates/design-template.md](templates/design-template.md);
- [templates/tasks-template.md](templates/tasks-template.md);
- [templates/pull-request-template.md](templates/pull-request-template.md);
- [templates/adr-template.md](templates/adr-template.md);
- [templates/adr-index-template.md](templates/adr-index-template.md);
- [templates/agents-root-template.md](templates/agents-root-template.md);
- [templates/openspec-agents-template.md](templates/openspec-agents-template.md).

Удаляйте неприменимые placeholders только после того, как команда действительно рассмотрела соответствующий вопрос.

## Checklists

Checklists используются в контрольных точках процесса:

- [checklists/spec-review-checklist.md](checklists/spec-review-checklist.md) — до implementation;
- [checklists/pr-review-checklist.md](checklists/pr-review-checklist.md) — во время PR Review;
- [checklists/archive-checklist.md](checklists/archive-checklist.md) — до, во время и после archive.

## Сквозной пример

Завершённый пример реального change:

- [examples/01-simple-feature/add-request-withdrawal/README.md](examples/01-simple-feature/add-request-withdrawal/README.md) — подробный architectural change с platform audit и ADR;
- [examples/02-non-architectural-bugfix/fix-withdrawal-error-message/README.md](examples/02-non-architectural-bugfix/fix-withdrawal-error-message/README.md) — компактный bugfix без нового ADR.

Пример показывает:

- proposal и delta spec;
- подробный design;
- конфигурация приложения;
- platform audit через Kafka;
- tasks и PR description;
- master spec после archive;
- archived change;
- Design Harvest, ADR и traceability.

Рекомендуется сначала прочитать основной process, а затем пройти пример в порядке, указанном в его README.

## Короткий выбор

```text
Нужно понять lifecycle
→ 01-workflow.md

Нужно начать новую систему
→ processes/new-project.md

Нужно изменить legacy
→ processes/brownfield.md

Нужно исправить баг
→ processes/bugfix.md

Нужен готовый реальный пример
→ examples/01-simple-feature/add-request-withdrawal/README.md
```
