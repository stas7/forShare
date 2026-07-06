# 07. OpenSpec Commands и Ответственность Ролей

## Зачем нужен этот документ

OpenSpec process состоит не только из документов и review. На разных стадиях команда запускает CLI commands.

Для каждой команды должно быть понятно:

- кто отвечает за запуск;
- когда она запускается;
- кто проверяет результат;
- изменяет ли команда repository state;
- какое решение команда принимает после результата.

## Главное правило

```text
Read-only command
→ может запускать любой участник.

State-changing command
→ запускает назначенный Owner после необходимых approvals.
```

По умолчанию:

- Analyst владеет expected behavior и проверяет specs;
- Developer / Change Owner запускает state-changing OpenSpec commands;
- QA проверяет Scenarios, testability и post-archive behavior;
- Reviewer независимо проверяет artifacts, validation output и archive result.

CLI command не заменяет review, QA sign-off или business decision.

## Типы Команд

### Read-Only

Эти команды не должны изменять OpenSpec artifacts:

```text
openspec list
openspec list --specs
openspec show <item-name>
openspec status --change <change-name>
openspec instructions <artifact> --change <change-name>
openspec validate <change-name> --strict
openspec validate --all --strict
openspec doctor
openspec context
openspec view
```

Любой участник может запустить их для независимой проверки.

### State-Changing

Эти команды создают или перемещают artifacts:

```text
openspec init <path>
openspec update <path>
openspec new change <change-name>
openspec archive <change-name> --yes
```

Их запускает назначенный Owner после согласования scope и условий выполнения.

Перед использованием `init` или `update` команда должна проверить локальный `openspec --help`: эти commands относятся к настройке repository, а не к обычному lifecycle каждого change.

## Command Matrix

| Стадия | Команда | Кто запускает | Кто проверяет результат | State |
|---|---|---|---|---|
| Первичная настройка repository | `openspec init <path>` | Repository maintainer / Developer | Reviewer | Изменяет |
| Обновление OpenSpec instructions | `openspec update <path>` | Repository maintainer / Developer | Reviewer | Изменяет |
| Explore | `openspec list` | Любая роль | Участник, который исследует change | Read-only |
| Explore | `openspec list --specs` | Analyst / Developer / QA | Сам участник | Read-only |
| Explore | `openspec show <item-name>` | Любая роль | Сам участник | Read-only |
| Explore | `openspec context` | Developer / Reviewer | Сам участник | Read-only |
| Explore / Health Check | `openspec doctor` | Developer / Reviewer | Reviewer | Read-only |
| Создание change | `openspec new change <change-name>` | Developer / Change Owner | Analyst + Reviewer | Изменяет |
| Подготовка artifacts | `openspec status --change <change-name>` | Developer / Change Owner | Все роли | Read-only |
| Подготовка artifact | `openspec instructions <artifact> --change <change-name>` | Owner соответствующего artifact | Reviewer | Read-only |
| Spec Review | `openspec validate <change-name> --strict` | Developer / Change Owner | Analyst + QA + Reviewer | Read-only |
| Implementation | `openspec status --change <change-name>` | Developer / Change Owner | Developer | Read-only |
| PR Review | `openspec validate <change-name> --strict` | Developer / Change Owner | Reviewer | Read-only |
| Pre-Archive | `openspec status --change <change-name>` | Developer / Change Owner | QA + Reviewer | Read-only |
| Pre-Archive | `openspec validate <change-name> --strict` | Developer / Change Owner | Analyst + QA + Reviewer | Read-only |
| Archive | `openspec archive <change-name> --yes` | Developer / Change Owner | Вся команда проверяет результат | Изменяет |
| Post-Archive | `openspec validate --all --strict` | Developer / Change Owner | Analyst + QA + Reviewer | Read-only |
| Post-Archive Health | `openspec doctor` | Developer / Reviewer | Reviewer | Read-only |

Распределение можно адаптировать, но Owner state-changing command должен быть назначен явно.

## 1. Первичная Настройка Repository

### Команда

```text
openspec init <path>
```

### Кто Запускает

Repository maintainer или Developer, отвечающий за структуру repository.

### Когда

Один раз при подключении OpenSpec к repository.

### Кто Проверяет

Reviewer проверяет созданные files, schema/configuration и Git diff.

Analyst и QA подтверждают, что structure позволяет находить capabilities и Scenarios.

### Важно

`init` не запускается для каждого change.

Если OpenSpec уже настроен, повторный `init` выполняется только после проверки ожидаемого impact.

## 2. Explore

### Команды

```text
openspec list
openspec list --specs
openspec show <item-name>
openspec view
```

### Кто Запускает

Любая роль.

Типичные случаи:

- Analyst просматривает existing specs и related changes;
- Developer ищет affected capabilities и active technical changes;
- QA ищет Scenarios и regression behavior;
- Reviewer проверяет conflicts и parallel changes.

### Результат

Команда понимает:

- какие active changes существуют;
- какие master specs затронуты;
- нет ли duplicate/conflicting change;
- какую capability нужно менять.

## 3. Проверка Context и Health

### Команды

```text
openspec context
openspec doctor
```

### Кто Запускает

Обычно Developer или Reviewer.

### Когда

- при первом знакомстве с repository;
- если OpenSpec root определяется неожиданно;
- если artifacts не связываются;
- перед большим change;
- после structural migration;
- после archive, если есть сомнения в relationship health.

### Кто Проверяет

Reviewer проверяет warnings и требует разобраться с ними до state-changing command.

## 4. Создание Active Change

### Команда

```text
openspec new change <change-name>
```

Дополнительные options используются только при необходимости:

```text
openspec new change <change-name> --description "<text>"
openspec new change <change-name> --goal "<text>"
openspec new change <change-name> --schema <schema-name>
```

### Кто Запускает

Developer / Change Owner.

### Что Делает Analyst

До запуска Analyst подтверждает:

- Why;
- What Changes;
- scope;
- change name с точки зрения смысла.

### Что Делает Reviewer

После запуска Reviewer проверяет:

- имя change;
- выбранную schema;
- отсутствие duplicate change;
- Git diff созданной structure.

### Что Делает QA

QA подтверждает, что change можно проверить отдельно и связать со Scenarios.

## 5. Подготовка Artifacts

### Status

```text
openspec status --change <change-name>
```

Developer / Change Owner запускает command после создания change и перед каждым gate.

Результат показывает completion artifacts по выбранной workflow schema.

### Instructions

```text
openspec instructions <artifact> --change <change-name>
```

Owner artifact запускает command перед подготовкой документа:

- Analyst — для proposal/spec artifacts;
- Developer — для design/tasks artifacts;
- QA и Reviewer — при независимой проверке требований к artifact.

Точное имя `<artifact>` берётся из `openspec status` или workflow schema. Не нужно угадывать artifact ID по filename.

### Важно

`instructions` выдаёт требования к artifact, но не подтверждает качество content. Owner всё равно отвечает за смысл, а Reviewer — за review.

## 6. Spec Review до Implementation

### Команда

```text
openspec validate <change-name> --strict
```

### Кто Запускает

Developer / Change Owner отвечает за основной запуск и прикладывает output к review.

Analyst, QA и Reviewer могут повторить command независимо.

### Что Проверяет Analyst

- proposal содержит `Why` и `What Changes`;
- delta соответствует expected behavior;
- master spec не изменена заранее.

### Что Проверяет QA

- Requirements и Scenarios структурно валидны;
- validation не заменяет testability review.

### Что Проверяет Reviewer

- command запущена для правильного change;
- используется `--strict`;
- warnings разобраны;
- output относится к текущей версии artifacts.

### Gate

Implementation начинается после:

```text
validation passed
+
Spec Review approved
+
go decision получено
```

Exit code `0` без анализа warnings не считается достаточным review.

## 7. Implementation

### Команда

```text
openspec status --change <change-name>
```

### Кто Запускает

Developer / Change Owner по мере выполнения tasks/artifacts.

### Когда

- после изменения design/spec;
- перед открытием PR;
- после существенного review comment;
- перед передачей в QA.

### Другие Роли

- Analyst повторно читает delta при изменении behavior;
- QA сверяет Scenarios и test plan;
- Reviewer проверяет, что artifact completion не подменяет content review.

## 8. PR Review

### Команды

```text
openspec status --change <change-name>
openspec validate <change-name> --strict
```

### Кто Запускает

Developer / Change Owner указывает commands/results в PR.

### Кто Проверяет

Reviewer независимо сверяет:

- proposal;
- delta spec;
- design;
- tasks;
- implementation;
- configuration/contracts;
- tests.

Analyst подключается при изменении behavior. QA проверяет build/test instructions и готовность QA.

OpenSpec validation не заменяет static analysis, automated tests и manual QA.

## 9. Перед Archive

### Commands

```text
openspec status --change <change-name>
openspec validate <change-name> --strict
```

### Кто Запускает

Developer / Change Owner.

### Кто Проверяет

- Analyst подтверждает финальную delta и expected behavior;
- QA подтверждает QA sign-off и regression;
- Reviewer подтверждает Design Harvest, ADR decision и отсутствие blocking comments;
- Developer проверяет закрытие tasks и фактический design.

### Gate

Archive command не запускается, пока:

- tasks не закрыты;
- QA sign-off не получен;
- Design Harvest не выполнен;
- ADR decision не зафиксирован;
- pre-archive validation не прошла.

## 10. Archive

### Команда

```text
openspec archive <change-name> --yes
```

### Кто Запускает

Developer / Change Owner.

### Что Делает CLI Автоматически

- применяет delta к master spec;
- создаёт capability spec, если требуется;
- перемещает active change в dated archive directory.

### Что Проверяет Команда Вручную

- Analyst — master spec и `Purpose`;
- QA — master Scenarios и сохранение проверенного behavior;
- Reviewer — archived structure, ADR links и traceability;
- Developer — command output, Git diff и полноту archive.

### Options

`--yes` отключает interactive confirmation, но не review.

`--skip-specs` не используется для functional change, которому нужно обновить master spec.

`--no-validate` не используется без отдельного exception decision.

## 11. После Archive

### Команды

```text
openspec validate --all --strict
openspec doctor
```

`doctor` запускается при необходимости relationship health check; итоговая strict validation обязательна.

### Кто Запускает

Developer / Change Owner запускает validation.

Developer или Reviewer запускает `doctor`, если archive изменил structure или есть warnings.

### Кто Проверяет

- Analyst проверяет нормативное behavior;
- QA проверяет Scenarios;
- Reviewer проверяет links/traceability;
- Developer проверяет diff и отсутствие active change.

После этого Developer / Change Owner выполняет Git commit и push по правилам repository.

OpenSpec CLI не создаёт Git commit и не выполняет push автоматически.

## 12. Cancelled или Superseded Change

У OpenSpec CLI не предполагается универсальная business command, которая решает, можно ли отменить change.

Если production behavior не изменилось, команда:

- фиксирует решение об отмене;
- закрывает PR/tasks;
- удаляет active change reviewed Git commit без применения delta;
- запускает `openspec validate --all --strict`.

Обычный archive не используется для имитации завершённого behavior.

Если behavior частично попало в production, создаётся recovery/rollback change.

## Независимая Validation

Хотя Developer / Change Owner отвечает за основной запуск, другие роли не обязаны доверять скопированному output.

Reviewer может повторить:

```text
openspec validate <change-name> --strict
```

QA может повторить validation перед sign-off.

Analyst может использовать `openspec show` и `openspec list --specs` для проверки итоговой structure.

## Работа с JSON Output

Для CI или автоматизированной проверки допустим `--json`:

```text
openspec list --json
openspec show <item-name> --json
openspec status --change <change-name> --json
openspec validate <change-name> --strict --json
openspec archive <change-name> --yes --json
openspec doctor --json
```

JSON mode не меняет responsibility roles. Он только делает output удобным для automation.

## Store Option

Если команда использует standalone OpenSpec store, commands получают:

```text
--store <id>
```

Owner обязан убедиться, что command направлена в правильный store. Reviewer проверяет store ID в evidence.

В обычном repository-root workflow `--store` не нужен.

## Что Команда Фиксирует в PR

Для command evidence достаточно:

```text
Command: openspec validate <change-name> --strict
Result: passed, warnings: 0
Commit SHA: <sha>
Executed by: <role/name>
```

Перед archive дополнительно фиксируется:

```text
Command: openspec status --change <change-name>
Result: all required artifacts complete

Command: openspec archive <change-name> --yes
Result: master spec updated, archived path created

Command: openspec validate --all --strict
Result: passed
```

## Ошибки, Которых Нужно Избегать

- Analyst запускает archive до QA sign-off.
- Developer запускает archive, не разобрав warning.
- Reviewer принимает screenshot без command/change/SHA.
- QA считает structural validation заменой functional testing.
- Команда использует `--skip-specs` для functional change.
- Команда использует `--no-validate` ради ускорения.
- Active change удаляется без решения и проверки production impact.
- После archive никто не проверяет master spec.
- CLI output считается approval роли.

## Короткая Шпаргалка

```text
Explore — любая роль
openspec list
openspec list --specs
openspec show <item-name>

Create — Developer / Change Owner
openspec new change <change-name>

Prepare — Owner artifact
openspec status --change <change-name>
openspec instructions <artifact> --change <change-name>

Review — Developer запускает, роли проверяют
openspec validate <change-name> --strict

Archive — Developer / Change Owner после approvals
openspec archive <change-name> --yes

Post-Archive — Developer запускает, команда проверяет
openspec validate --all --strict
```
