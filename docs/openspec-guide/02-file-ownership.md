# 02. Кто за какой artifact отвечает

Таблица использует общие роли. Конкретная команда может назначить разных людей внутри одной роли, но ответственность за результат должна оставаться явной.

## Обозначения

| Обозначение | Значение |
|---|---|
| Owner | Отвечает за подготовку и актуальность artifact или результата |
| Review | Проверяет и может оставить blocking comments |
| Consult | Участвует в подготовке и предоставляет обязательный контекст |
| Verify | Проверяет итог после выполнения операции |
| Read | Должен понимать результат, но не даёт обязательный approval |
| — | Обычно не участвует, если нет отдельной причины |

`Owner` не означает, что человек пишет документ один. Owner организует работу, закрывает замечания и отвечает за согласованность результата.

## Active Change

| Artifact / результат | Analyst | Developer | QA | Reviewer |
|---|---:|---:|---:|---:|
| `proposal.md` | Owner | Review | Review | Review |
| Delta `spec.md` | Owner | Review | Review | Review |
| `design.md` | Consult | Owner | Consult | Review |
| `tasks.md` | Consult | Owner | Consult | Review |
| Application configuration | Consult | Owner | Review | Review |
| API / message / schema contracts | Consult | Owner | Review | Review |
| Migration plan | Consult | Owner | Review | Review |
| Rollout / rollback plan | Consult | Owner | Review | Review |
| Observability plan | Consult | Owner | Review | Review |

### Пояснения

Аналитик является owner delta spec, потому что отвечает за ожидаемое поведение. Developer проверяет реализуемость, QA — проверяемость, Reviewer — conflicts и ограничения.

Developer является owner design и technical contracts. Если configuration определяет бизнес-поведение, Analyst обязательно участвует как Consult, а QA проверяет testability и environment differences.

Reviewer не переписывает artifact вместо Owner. Reviewer формулирует замечание и ожидаемый критерий исправления; Owner обновляет artifact и закрывает comment.

## Implementation and Testing

| Artifact / результат | Analyst | Developer | QA | Reviewer |
|---|---:|---:|---:|---:|
| Production code | Consult | Owner | Read | Review |
| Automated tests | Consult | Owner | Review | Review |
| Characterization tests | Consult | Owner | Review | Review |
| Test data | Consult | Consult | Owner | Read |
| Manual test plan | Consult | Consult | Owner | Read |
| Regression scope | Consult | Consult | Owner | Review |
| Manual QA evidence | Read | Consult | Owner | Read |
| QA sign-off | Read | Consult | Owner | Read |
| PR description | Review | Owner | Review | Review |
| Test expectation conflict decision | Owner behavior | Consult | Consult | Review |

### Test Expectations

Если production-код конфликтует с существующим test expectation:

- Developer фиксирует actual и ожидаемое поведение;
- Analyst определяет, меняется ли ожидаемое поведение;
- QA оценивает корректность и строгость test;
- Reviewer проверяет согласованность решения с OpenSpec;
- test expectation изменяется только после явного решения и обновления active change, если меняется поведение.

## Design Harvest

| Artifact / результат | Analyst | Developer | QA | Reviewer |
|---|---:|---:|---:|---:|
| Проверка финального наблюдаемое поведение | Owner | Consult | Verify | Review |
| Обновление `design.md` по факту реализации | Consult | Owner | Consult | Review |
| Решение о необходимости ADR | Consult | Owner | Consult | Review |
| ADR | Consult | Owner | Read | Review |
| Architecture docs | Consult | Owner | Read | Review |
| Testing / regression docs | Consult | Consult | Owner | Review |
| Follow-up technical debt | Consult | Owner | Consult | Review |

Design Harvest выполняется командой вручную. OpenSpec CLI не извлекает решения из `design.md` и не создаёт ADR автоматически.

## Archive

| Artifact / операция | Analyst | Developer | QA | Reviewer |
|---|---:|---:|---:|---:|
| Готовность delta spec к archive | Owner | Review | Verify | Review |
| Закрытие `tasks.md` | Consult | Owner | Verify | Review |
| Pre-archive validation | Read | Owner | Read | Review result |
| Запуск archive command | Read | Owner | Read | Read |
| Master spec после archive | Owner behavior | Verify structure | Verify Scenarios | Review |
| `Purpose` новой capability | Owner | Consult | Consult | Review |
| Archived change | Verify context | Owner | Verify evidence | Review |
| ADR source links | Consult | Owner | Read | Review |
| ADR traceability index | Read | Owner | Read | Review |
| Post-archive validation | Read | Owner | Read | Review result |
| Git commit / push archive result | Read | Owner | Read | Review scope |

### Что выполняется автоматически

После ручного запуска archive command OpenSpec CLI автоматически:

- применяет delta к master spec;
- создаёт capability spec, если она отсутствует;
- перемещает active change в dated archive directory.

### Что остаётся ответственностью команды

Команда вручную:

- выполняет Design Harvest;
- создаёт и обновляет ADR;
- проверяет master spec;
- заполняет содержательный `Purpose`;
- обновляет traceability index;
- проверяет archived links;
- выполняет post-archive validation;
- делает Git commit и push.

Подробный порядок описан в [checklists/archive-checklist.md](checklists/archive-checklist.md).

## Master Spec

Master spec — постоянный источник актуального наблюдаемое поведение.

- Analyst отвечает за смысл и непротиворечивость behavior.
- Developer проверяет соответствие реализованным contracts и ограничениям.
- QA проверяет, что Scenarios соответствуют фактически проверенному поведению.
- Reviewer проверяет, что детали реализации не попали в master spec без contract reason.

Master spec не используется как журнал изменений. История Why, design alternatives, tasks и migration остаётся в archived change.

## ADR и Traceability

Developer обычно является owner ADR как технического решения.

Analyst участвует, если решение влияет на наблюдаемое поведение или business constraints. QA участвует, если решение создаёт постоянные regression или operability requirements. Reviewer даёт architecture approval.

Каждый archived change:

- связывается с ADR, если появилось долговременное архитектурное решение;
- либо отмечается как `non-architectural` в traceability index.

## Простое правило

- Analyst отвечает за смысл и ожидаемое поведение.
- Developer отвечает за техническое решение, implementation и lifecycle artifacts.
- QA отвечает за testability, evidence и regression.
- Reviewer отвечает за независимую проверку качества, contracts и ограничений.

Подробное распределение между людьми внутри команды 1 аналитик / 2 разработчика / 2 QA описано в [03-team-composition.md](03-team-composition.md).
