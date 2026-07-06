# 03. Состав команды и конкретные роли

Этот guide рассчитан на пилотную команду:

```text
1 аналитик
2 разработчика
2 тестировщика
```

OpenSpec не требует именно такого состава. Но для первого внедрения важно явно договориться, кто за что отвечает.

## Состав команды

| Участник | Рабочая роль в OpenSpec | Главная ответственность |
|---|---|---|
| Аналитик | Analyst / Requirement Owner | Смысл задачи, scope, требования, scenarios, acceptance criteria |
| Разработчик 1 | Developer / Change Owner | Ведение change, `design.md`, `tasks.md`, реализация, синхронизация artifacts |
| Разработчик 2 | Developer Reviewer / Architecture Reviewer | Проверка технического подхода, архитектурных ограничений, PR-review |
| Тестировщик 1 | QA Owner | Тест-дизайн, manual test plan, regression scope, QA sign-off |
| Тестировщик 2 | QA Reviewer / Exploratory QA | Независимая проверка сценариев, edge cases, exploratory testing |

## Как задача проходит через этих людей

```text
Аналитик
  ↓
формулирует задачу, proposal и scenarios

Разработчик 1
  ↓
пишет design, tasks и реализует изменение

Разработчик 2
  ↓
проверяет design до реализации и PR после реализации

Тестировщик 1
  ↓
готовит тест-план и выполняет основное тестирование

Тестировщик 2
  ↓
проверяет полноту тестов, edge cases и делает независимую проверку
```

## Конкретные обязанности по этапам

| Этап | Аналитик | Разработчик 1 | Разработчик 2 | Тестировщик 1 | Тестировщик 2 |
|---|---|---|---|---|---|
| Explore | Owner по бизнес-смыслу | Помогает исследовать код и ограничения | Консультирует по архитектуре | Задаёт вопросы по проверяемости | Ищет неочевидные кейсы |
| Proposal | Owner | Review | Review | Review | Review |
| Specification | Owner | Review | Review | Co-owner по scenarios | Review edge cases |
| Design | Consult | Owner | Architecture review | Read / задаёт вопросы | Read |
| Tasks | Consult | Owner | Review | Добавляет QA tasks | Review QA tasks |
| Pre-code review | Утверждает смысл | Защищает подход | Утверждает архитектуру | Утверждает проверяемость | Проверяет полноту |
| Implementation | Consult | Owner | Consult / pair review | Готовит тесты | Готовит exploratory ideas |
| PR Review | Проверяет смысловые изменения | Owner PR | Main reviewer | Проверяет тестируемость | Проверяет риски |
| QA | Consult | Fix defects | Consult | Owner QA | Independent QA |
| Design Harvest | Проверяет, не потерян ли бизнес-смысл | Owner | Проверяет ADR / architecture docs | Проверяет QA lessons | Проверяет regression lessons |
| Archive | Review | Owner | Review | QA sign-off | Review |

## Что значит Change Owner

Change Owner — это человек, который следит, чтобы задача не развалилась на несвязанные части.

В пилотной команде Change Owner обычно Разработчик 1.

Он отвечает, чтобы были согласованы:

```text
proposal.md
spec.md
design.md
tasks.md
code
tests
archive
```

Change Owner не обязан единолично всё писать, но он отвечает за целостность change.

## Что значит QA Owner

QA Owner отвечает не только за тестирование после разработки.

Он должен подключаться раньше — на этапе Specification Review.

Главный вопрос QA Owner:

> Сможем ли мы проверить это требование?

Если scenario невозможно проверить, его нужно переписать до начала реализации.

## Что значит Developer Reviewer

Developer Reviewer — это не просто человек, который смотрит стиль кода.

Он проверяет:

- не нарушены ли архитектурные ограничения;
- не появился ли скрытый scope;
- соответствует ли код `design.md`;
- не нужно ли вынести новое правило в ADR;
- не останется ли после archive важное знание только в PR-комментариях.

## Минимальный порядок согласования

Для первой пилотной задачи используйте такой порядок:

```text
1. Analyst готовит proposal + spec.
2. QA Owner проверяет scenarios.
3. Developer 1 готовит design + tasks.
4. Developer 2 проверяет design.
5. Команда даёт короткий go/no-go до реализации.
6. Developer 1 реализует.
7. Developer 2 делает PR-review.
8. QA Owner тестирует.
9. QA Reviewer делает независимую проверку.
10. Developer 1 выполняет Design Harvest и Archive.
```

## Главное правило для команды из 5 человек

Не все должны писать все документы.

Но каждый должен смотреть свой участок ответственности до того, как задача перейдёт дальше.
