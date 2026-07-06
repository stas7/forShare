# 04. Инструкции для AI-агентов

## Зачем нужны AGENTS.md

OpenSpec описывает изменение продукта, но сам по себе не задаёт все правила работы AI-агента с репозиторием.

`AGENTS.md` фиксирует обязательные ограничения:

- что нужно прочитать до изменения;
- когда требуется OpenSpec change;
- какие проверки обязательны;
- как обращаться с существующими тестами;
- где хранить архитектурные решения;
- какие локальные инструкции действуют в отдельных директориях.

## Иерархия инструкций

Рекомендуемая структура:

```text
AGENTS.md
openspec/AGENTS.md
<module>/AGENTS.md
```

Корневой `AGENTS.md` содержит правила для всего репозитория.

`openspec/AGENTS.md` уточняет работу с proposal, delta spec, design, tasks и archive.

Локальный `AGENTS.md` описывает ограничения конкретного модуля. Он может усиливать общие правила, но не должен ослаблять обязательные ограничения корневой инструкции.

Не нужно копировать один и тот же текст во все файлы. Корневая инструкция задаёт invariants, локальная — только дополнительный контекст.

## Обязательная защита test expectations

Существующий passing test фиксирует ожидаемое или ранее подтверждённое поведение.

Если новый production-код приводит к failing test, агент не должен автоматически считать тест устаревшим.

До явного решения человека запрещено ради успешного прохождения нового кода:

- изменять или удалять assertions;
- ослаблять expected result;
- пропускать test;
- заменять точную проверку на более общую;
- менять fixtures, mocks или golden files так, чтобы скрыть конфликт;
- снижать coverage threshold;
- исключать файл из test suite или coverage;
- скрывать ошибку с помощью broad exception handling или необоснованного retry.

Агент должен:

1. Остановить автоматическое исправление конфликта.
2. Показать failing test.
3. Показать actual и ожидаемое поведение.
4. Объяснить наиболее вероятную причину: дефект production-код, намеренное изменение requirement или дефект теста.
5. Описать варианты решения и влияние на совместимость и OpenSpec.
6. Дождаться явного решения человека перед изменением test expectation.

Если поведение действительно меняется, сначала обновляется active OpenSpec change и delta spec. После review можно обновить test expectation.

Техническое исправление test harness допустимо без отдельного согласования, только если доказуемо не меняет проверяемое поведение и строгость assertions.

## Пример конфликта

```text
Requirement:
Заявку нельзя отозвать после начала согласования.

Existing test:
Ожидает 409 для заявки в статусе ApprovalStarted.

New implementation:
Возвращает 200 и переводит заявку в Withdrawn.
```

Неправильно:

```text
Изменить expected status с 409 на 200, чтобы test прошёл.
```

Правильно:

```text
1. Зафиксировать actual: 200 / Withdrawn.
2. Зафиксировать expected: 409 / статус не меняется.
3. Сверить master spec и active change.
4. Сообщить о конфликте человеку.
5. Получить решение: исправлять production-код или менять requirement.
6. Синхронно обновить spec, code и tests.
```

## OpenSpec и AGENTS.md

Для functional change корневая инструкция должна требовать:

```text
прочитать project context
→ прочитать затронутую master spec
→ прочитать active change
→ реализовать по reviewed artifacts
→ проверить proposal, spec, design, tasks, code и tests
```

Future behavior не записывается напрямую в master spec. Оно сначала описывается в delta spec активного change и попадает в master spec при archive.

Archived change считается historical snapshot. Агент не должен менять его semantic content задним числом. Новое behavior оформляется новым change, а изменение долговременного решения — новым ADR с supersession.

## Минимальная проверка

`AGENTS.md` должен перечислять реальные команды проекта, а не абстрактную фразу «запустить тесты».

OpenSpec validation зависит от стадии lifecycle:

```text
Active change: openspec validate <change-name> --strict
После archive: openspec validate --all --strict
Static analysis: <команда проекта>
Automated tests: <команда проекта>
Manual QA: <затронутые Scenarios и целевая среда>
```

Первая команда проверяет delta и структуру конкретного active change. Вторая проверяет итоговые master specs после применения delta и перемещения change в archive.

Команды из другого проекта нельзя копировать без проверки. Например, Flutter-команды относятся только к Flutter-проекту.

## ADR lifecycle и traceability

ADR создаётся для долговременного архитектурного решения, а не для каждого change.

Рекомендуемый формат ADR:

```text
Status
Context
Decision
Consequences
Alternatives Considered
Источники
```

Раздел `Источники` связывает ADR с archived change и его `design.md`.

Правила lifecycle:

- ADR получает последовательный номер, который не переиспользуется;
- принятое решение не переписывается задним числом;
- изменение решения оформляется новым ADR;
- старый ADR получает статус `Superseded` и ссылку на новый;
- новый ADR ссылается на заменённый;
- archived change связывается с ADR либо отмечается как `non-architectural`.

Пример traceability table:

| Archived change | ADR | Классификация |
|---|---|---|
| `add-request-withdrawal` | `ADR-001` | Architectural |
| `fix-withdrawal-message` | — | Non-architectural |

Traceability нужна, чтобы после archive было понятно, где сохранено архитектурное решение и почему change не требовал ADR.

## Что не переносить между проектами автоматически

Не следует превращать в универсальные правила:

- имя конкретного API schema;
- команды конкретного framework;
- требования к определённому design tool;
- правила startup или authentication одного приложения;
- конкретные fixture values;
- имена внутренних сервисов.

Такие правила размещаются в локальном `AGENTS.md` соответствующего проекта или модуля.

## Где посмотреть пример

Сквозной пример находится в:

```text
docs/openspec-guide/examples/01-simple-feature/add-request-withdrawal/
```

В нём корневой `AGENTS.md` задаёт обязательные правила, а `openspec/AGENTS.md` уточняет работу с OpenSpec artifacts.
