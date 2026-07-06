# Checklist: Archive

Archive состоит из трёх стадий:

```text
подготовка командой
→ выполнение команды OpenSpec archive
→ проверка результата командой
```

Важно различать действия команды и автоматические действия OpenSpec CLI.

## 1. Перед archive — команда проверяет вручную

```text
[ ] Реализация завершена.
[ ] PR comments обработаны.
[ ] PR approval получен.
[ ] QA sign-off получен.
[ ] Manual QA затронутых Scenarios выполнен.
[ ] Regression выполнен.
[ ] Static analysis и automated tests выполнены.
[ ] Test expectations изменены только после явного согласования нового поведения.
[ ] proposal.md отражает финальные Why и What Changes.
[ ] Delta spec отражает финальное наблюдаемое поведение.
[ ] design.md соответствует фактической реализации.
[ ] tasks.md полностью закрыт.
[ ] Design Harvest выполнен.
[ ] Определено, какие решения из design.md должны попасть в ADR.
[ ] ADR создан или обновлён, если появилось долговременное архитектурное решение.
[ ] Change классифицирован как architectural или non-architectural.
[ ] Новые постоянные правила подготовлены для соответствующих docs.
[ ] Временные детали оставлены в design.md и других artifacts change.
```

На этой стадии команда не изменяет master spec вручную для описания future behavior. Источником изменения остаётся delta spec active change.

## 2. Перед archive — команда запускает validation вручную

Команда выполняет:

```text
openspec validate <change-name> --strict
```

Команда вручную анализирует warnings и errors. Нельзя выполнять archive с непонятным warning только потому, что validation завершилась с exit code `0`.

Проверяется:

```text
[ ] Validation active change завершилась успешно.
[ ] Все warnings разобраны и устранены либо явно обоснованы.
[ ] Обязательные sections proposal присутствуют.
[ ] Delta operations ADDED / MODIFIED / REMOVED корректны.
[ ] Requirements и Scenarios имеют правильную структуру.
```

## 3. Команда запускает archive вручную

После успешной проверки команда выполняет:

```text
openspec archive <change-name> --yes
```

`--yes` отключает только interactive confirmation. Он не отменяет ответственность команды за review результата.

Не используйте `--skip-specs` для functional change: этот option не применит delta к master spec.

Не используйте `--no-validate` без отдельного обоснованного решения команды.

## 4. Что OpenSpec CLI делает автоматически

При обычном functional archive OpenSpec CLI автоматически:

1. Проверяет структуру change.
2. Читает delta specs из active change.
3. Применяет `ADDED`, `MODIFIED` и `REMOVED` requirements к соответствующим master specs.
4. Создаёт master spec capability, если её ещё не было.
5. Перемещает change из active changes в dated archive directory.
6. Сохраняет в archived change файлы `proposal.md`, `design.md`, `tasks.md` и delta specs.

Пример результата:

```text
openspec/changes/add-request-withdrawal/
→
openspec/changes/archive/2026-07-06-add-request-withdrawal/

openspec/changes/add-request-withdrawal/specs/approval-requests/spec.md
→ delta применяется к
openspec/specs/approval-requests/spec.md
```

Если capability создаётся впервые, CLI может сформировать placeholder `Purpose`. Команда обязана проверить и заменить `TBD` содержательным описанием.

## 5. Что OpenSpec CLI не делает автоматически

Archive command не подтверждает и не выполняет автоматически:

- PR approval;
- QA sign-off;
- regression;
- production verification;
- Design Harvest;
- выбор архитектурных решений для ADR;
- создание или обновление ADR;
- обновление ADR traceability index;
- проверку business correctness master spec;
- заполнение содержательного `Purpose` новой capability;
- обновление architecture docs и testing docs;
- проверку ссылок на новый archived path;
- commit и push результата archive.

За эти действия отвечает команда.

## 6. После archive — команда проверяет результат вручную

```text
[ ] Active change directory больше не существует.
[ ] Создан ожидаемый dated archived change.
[ ] Archived change содержит proposal.md, design.md, tasks.md и delta specs.
[ ] Master spec содержит финальное ожидаемое поведение.
[ ] В master spec нет случайных детали реализации из design.md.
[ ] Purpose новой capability заполнен и не содержит TBD.
[ ] Все Requirements и Scenarios сохранились корректно.
[ ] ADR содержит ссылку на archived change и archived design.md.
[ ] ADR добавлен в traceability index.
[ ] Change без ADR отмечен как non-architectural.
[ ] Если решение заменено, создан новый ADR, а старый получил status Superseded.
[ ] Постоянные architecture/testing rules обновлены.
[ ] Git diff содержит только ожидаемые изменения archive.
```

## 7. После archive — команда запускает итоговую validation вручную

Команда выполняет:

```text
openspec validate --all --strict
```

Эта команда проверяет итоговые master specs после применения delta.

После неё команда также запускает специфичный для проекта проверки, если archive сопровождался дополнительными изменениями:

```text
Static analysis: <команда проекта>
Automated tests: <команда проекта>
Documentation validation: <команда проекта, если есть>
```

## 8. Git commit и push выполняются командой вручную

После проверки diff команда создаёт commit и выполняет push по правилам репозитория.

OpenSpec archive сам по себе не создаёт Git commit и не отправляет изменения в remote repository.

Финальный результат:

```text
master spec = актуальное наблюдаемое поведение
ADR = извлечённые долговременные архитектурные решения
archived change = полная история proposal, delta spec, design и tasks
```
## 9. Archived Change становится Historical Snapshot

После успешного archive:

```text
[ ] Semantic content archived proposal не изменяется задним числом.
[ ] Archived delta spec не используется для исправления master spec.
[ ] Archived design не переписывается под новое architecture decision.
[ ] Completed tasks не меняются для имитации другой истории.
[ ] Новое behavior оформляется новым change.
[ ] Новое architecture decision оформляется новым ADR.
[ ] Нессемантическая правка имеет отдельный commit с объяснением.
```

Допустимая несемантическая правка:

```text
Исправить broken link на перемещённый ADR без изменения Decision.
```

Недопустимая правка:

```text
Изменить archived Requirement, чтобы он совпал с новым production behavior.
```

Во втором случае создаётся новый change, который обновляет master spec обычным delta/archive process.
