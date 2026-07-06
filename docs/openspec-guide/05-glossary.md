# 05. Glossary и правила языка

## Зачем нужен glossary

Руководство написано на русском языке, но OpenSpec и разработка используют устойчивые английские термины.

Цель glossary — избежать двух крайностей:

- не переводить filenames, syntax и contracts так, что они перестают совпадать с инструментами;
- не превращать обычные объяснения в смесь русского и английского без необходимости.

## Основное правило

```text
Имена artifacts, syntax, roles, lifecycle stages и общепринятые technical terms
→ сохраняются в исходном виде.

Обычный описательный текст
→ пишется на русском языке.
```

Если английское слово не является identifier, filename, keyword или устойчивым термином команды, предпочтительно использовать русское пояснение.

## OpenSpec Artifacts

Эти названия сохраняются:

| Термин | Значение |
|---|---|
| `change` | Отдельное изменение, проходящее OpenSpec lifecycle |
| `proposal.md` | Зачем нужен change и что изменяется |
| `design.md` | Техническое решение |
| `tasks.md` | Проверяемый план работ |
| `spec.md` | Requirements и Scenarios |
| `master spec` | Актуальное нормативное поведение capability |
| `delta spec` | Изменение master spec внутри active change |
| `active change` | Change до archive |
| `archived change` | История change после archive |
| `artifact` | Документ или другой результат процесса |
| `capability` | Область поведения системы, которой соответствует spec |

Имена файлов всегда пишутся точно: `proposal.md`, а не «файл предложения».

## Обязательные Proposal Headings

Заголовки сохраняются в форме, ожидаемой OpenSpec:

```text
Why
What Changes
Out of Scope
Affected Areas
Risks and Constraints
Success Criteria
```

В пояснении можно писать «почему change нужен» и «что изменяется», но heading в template не переводится.

## Specification Syntax

Не переводятся:

```text
Requirement
Scenario
GIVEN
WHEN
THEN
AND
SHALL
ADDED
MODIFIED
REMOVED
```

Текст после keyword пишется по-русски.

Правильно:

```md
### Requirement: Отзыв заявки

Система SHALL разрешать автору отозвать заявку до начала согласования.

#### Scenario: Автор отзывает заявку

- GIVEN заявка находится в статусе `Submitted`
- WHEN автор отзывает заявку
- THEN система изменяет статус заявки на `Withdrawn`
```

Неправильно:

```md
### Requirement: Request Withdrawal

The system SHALL allow the author to withdraw the request.
```

## Lifecycle Stages

Названия stages и review gates можно сохранять:

| Термин | Пояснение |
|---|---|
| Explore | Исследование задачи и текущего состояния |
| Specification | Описание нормативного поведения |
| Design | Подготовка технического решения |
| Implementation | Реализация |
| Spec Review | Проверка proposal и delta spec до реализации |
| PR Review | Проверка implementation и всей цепочки artifacts |
| QA | Проверка behavior и regression |
| Design Harvest | Извлечение постоянных знаний перед archive |
| Archive | Применение delta и сохранение истории change |

В обычном предложении можно использовать русский глагол:

```text
выполнить review
архивировать change
проверить реализацию
```

Commands и options не переводятся:

```text
openspec validate <change-name> --strict
openspec archive <change-name> --yes
openspec validate --all --strict
```

## Roles и Responsibility

Сохраняются названия ролей:

```text
Analyst
Developer
QA
Reviewer
Owner
```

При первом употреблении рекомендуется дать русское пояснение.

Пример:

```text
Analyst — Owner ожидаемого поведения.
Developer — Owner технического решения и реализации.
```

Статусы участия сохраняются:

```text
Owner
Review
Consult
Verify
Read
```

## ADR

Сохраняются:

```text
ADR
Architecture Decision Record
Status
Context
Decision
Consequences
Alternatives Considered
Proposed
Accepted
Deprecated
Superseded
Non-architectural
```

Обычный текст ADR пишется по-русски. Заголовки формата Nygard сохраняются.

`design.md` целиком не переносится в ADR. В ADR фиксируются долговременное архитектурное решение, причины и последствия.

## Общепринятые Technical Terms

Следующие термины допустимо сохранять, если они привычны команде и точнее русского аналога:

```text
API
UI
HTTP
Kafka
JSON
schema
payload
topic
permission
role mapping
feature flag
commit
push
pull request / PR
CI
unit test
integration test
contract test
regression
retry
DLQ
outbox
idempotency
deduplication
correlation ID
rollout
rollback
production
```

Не нужно искусственно переводить identifiers:

```text
APPROVAL_REQUEST_WITHDRAWN
approval-request.withdraw-own
RequestStatusTransitionService
PLATFORM_AUDIT_TOPIC
```

## Что переводится в описательном тексте

Предпочтительные формулировки:

| Не рекомендуется без причины | Предпочтительно |
|---|---|
| current behavior | текущее поведение |
| expected behavior | ожидаемое поведение |
| observable behavior | наблюдаемое поведение |
| technical solution | техническое решение |
| business rule | бизнес-правило |
| application configuration | конфигурация приложения |
| platform capability | возможность платформы |
| implementation details | детали реализации |
| architecture decision | архитектурное решение |
| sensitive data | чувствительные данные |
| source code | исходный код |
| error behavior | поведение при ошибке |

Если английская форма является названием heading или поля contract, она сохраняется.

## Scope и Out of Scope

`scope` и `out of scope` допустимы как устойчивые process terms.

При необходимости поясняйте:

```text
scope — границы change;
out of scope — то, что намеренно не входит в change.
```

Не используйте одновременно несколько русских вариантов «объём», «охват», «границы» в одном документе, если речь идёт об одном и том же scope.

## Platform Terminology

В описательном тексте предпочтительно:

```text
возможности платформы
конфигурация приложения
граница ответственности приложения и платформы
платформенный модуль аудита
```

В headings и diagrams допустимы устойчивые названия:

```text
Application Business Logic
Application-Owned Configuration
Platform Capabilities
```

Названия реальных modules, SDK и configuration keys не переводятся.

## Evidence и Checks

Вместо неопределённого `evidence` предпочтительно писать конкретно:

- результаты tests;
- CI link;
- QA evidence;
- фактический response;
- log по correlation ID;
- screenshot;
- payload;
- результат validation.

Если `evidence` используется как process term, glossary должен быть известен команде.

## Правила для Examples

В примерах:

- headings и syntax сохраняются;
- текст Requirements и Scenarios пишется по-русски;
- identifiers и contracts не переводятся;
- example commands должны быть реально применимы к состоянию change;
- вымышленные test results явно помечаются как моделируемые;
- обычные объяснения пишутся по-русски.

## Как добавлять новый термин

Перед добавлением английского слова проверьте:

1. Это filename, command, identifier или syntax keyword?
2. Это устойчивый technical/process term команды?
3. Русский вариант менее точен или создаёт неоднозначность?
4. Термин уже используется одинаково в других документах?

Если ответы отрицательные, используйте русский текст.

Если термин важен для всего руководства, добавьте его в glossary и используйте последовательно.
