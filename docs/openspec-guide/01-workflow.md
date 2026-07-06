# 01. Общий workflow

## Для какой команды рассчитан процесс

Базовый пилотный состав:

```text
1 аналитик
2 разработчика
2 тестировщика
```

В этом составе:

- аналитик отвечает за смысл, scope и requirements;
- разработчик 1 ведёт change как Change Owner;
- разработчик 2 проверяет design и PR как Developer Reviewer;
- тестировщик 1 отвечает за test plan и QA sign-off как QA Owner;
- тестировщик 2 проверяет edge cases и делает независимую проверку как QA Reviewer.

Подробно роли описаны в `03-team-composition.md`.

## Жизненный цикл задачи

```text
1. Идея / задача / баг
2. Explore
3. Proposal
4. Specification
5. Design
6. Tasks
7. Review до реализации
8. Implementation
9. PR Review
10. QA
11. Design Harvest
12. Archive
```

Подробная matrix «стадия → command → роль → проверяющий» находится в [07-openspec-commands.md](07-openspec-commands.md).

## Explore

Команда выясняет, что именно нужно сделать: обсуждает задачу, смотрит существующий код, ограничения и текущие specs.

## Proposal

`proposal.md` отвечает на вопросы: зачем делаем, что входит в scope, что не входит в scope, какие области затронуты, какие есть риски.

## Specification

Specification описывает наблюдаемое поведение системы: что должна делать система. В specification не пишем классы, методы и внутренние детали реализации.

## Design

`design.md` описывает технический подход: как собираемся реализовывать изменение, какие есть ограничения, риски, интеграции и миграции.

## Tasks

`tasks.md` разбивает работу на небольшие понятные шаги.

## Review до реализации

Перед реализацией команда смотрит proposal, spec, design и tasks. Цель — договориться о смысле и техническом подходе заранее.

## Implementation

Разработчик реализует задачу. Если во время реализации меняется поведение или подход, сначала обновляются OpenSpec-файлы.

## PR Review

Reviewer проверяет цепочку: proposal → spec → design → tasks → code → tests.

## QA

QA проверяет сценарии из specification, выполняет ручные проверки и regression.

## Design Harvest

Перед archive команда сравнивает delta spec, `design.md` и фактическую реализацию.

Знания распределяются так:

```text
Observable behavior из delta spec
→ master spec при archive

Долговременные архитектурные решения из design.md
→ ADR или architecture docs вручную

Полный технический контекст, альтернативы и временные детали
→ archived design.md
```

`design.md` целиком не переносится ни в master spec, ни в ADR.

Команда вручную создаёт или обновляет ADR, traceability index, architecture docs и testing docs. OpenSpec CLI не выполняет Design Harvest автоматически.

## Archive

Сначала команда вручную проверяет готовность change и запускает:

```text
openspec validate <change-name> --strict
openspec archive <change-name> --yes
```

OpenSpec CLI автоматически применяет delta к master spec и перемещает change в dated archive directory.

После archive команда вручную проверяет master spec, archived change, `Purpose`, ADR links и traceability, затем запускает:

```text
openspec validate --all --strict
```

Archive CLI не выполняет QA, Design Harvest, создание ADR, Git commit или push.

Подробные действия до, во время и после archive описаны в [checklists/archive-checklist.md](checklists/archive-checklist.md).
## Неизменяемость Archived Change

Archived change является историческим snapshot состояния change на момент archive.

После archive не переписываются задним числом:

- Why и What Changes;
- Requirements и Scenarios delta spec;
- technical decisions в `design.md`;
- completed state `tasks.md`;
- факты implementation, QA и rollout.

Если поведение нужно изменить, создаётся новый OpenSpec change. Если изменяется долговременное архитектурное решение, создаётся новый ADR, а предыдущий получает status `Superseded`.

Допустимы только несемантические исправления, например broken link или явная опечатка. Такое исправление выполняется отдельным commit с объяснением, почему historical meaning не изменилось.

Master spec исправляется через новый delta change, а не редактированием archived delta.
## Отменённый, Зависший или Superseded Change

Не каждый active change доходит до implementation и archive.

Команда сначала определяет фактическое состояние:

```text
1. Production behavior не менялось.
2. Code/configuration изменены, но не выпущены.
3. Изменение частично или полностью попало в production.
4. Change заменён другим change.
5. Change устарел и больше не соответствует master spec.
```

Обычный successful archive используется только для change, который действительно реализован, проверен и должен обновить master spec.

### Production Behavior не Менялось

Если implementation не начиналась или не была выпущена:

- зафиксируйте причину отмены в task, issue или PR;
- укажите, кто и когда принял решение;
- закройте связанные PR/tasks;
- убедитесь, что delta не применялась к master spec;
- удалите active change отдельным reviewed Git commit;
- не запускайте обычный `openspec archive`, потому что behavior не реализовано.

Минимальная запись:

```text
Change status: Cancelled
Reason: <почему change больше не нужен>
Production impact: none
Replacement: <ссылка на новый change или не применимо>
Decision owner/date: <кто и когда>
```

OpenSpec CLI не принимает business decision об отмене автоматически. Закрытие выполняется командой по правилам repository.

### Code Изменён, но не Выпущен

Если code/configuration существует только в branch или незамерженном PR:

- закройте или переработайте PR;
- удалите/revert незавершённую implementation безопасным Git change;
- подтвердите, что shared environments и production не изменены;
- сохраните причину отмены;
- после review удалите active OpenSpec change без применения delta.

Не используйте archive только ради очистки `openspec/changes/`.

### Behavior Частично Попало в Production

Такой change нельзя считать просто отменённым.

Команда должна:

1. Зафиксировать фактическое production behavior.
2. Определить целевое behavior: завершить rollout или выполнить rollback.
3. Создать recovery/rollback OpenSpec change.
4. Описать current, expected и preserved behavior.
5. Синхронизировать code, configuration, tests и specs.
6. Выполнить QA, regression и production verification.
7. Архивировать recovery change обычным процессом.

Удаление исходного active change без recovery создаст расхождение между production и master spec.

### Superseded Change

Если один active change заменён другим:

- новый change ссылается на заменённый;
- в старом change фиксируется причина supersession;
- переносимые requirements/design decisions явно переносятся в новый change;
- устаревшая delta не применяется к master spec;
- старый change закрывается по repository policy;
- связанные PR/issues получают ссылку на replacement.

Пример:

```text
Change status: Superseded
Superseded by: replace-withdrawal-workflow
Reason: первоначальный scope заменён единым workflow change
Production impact: none
```

`Superseded` для change не равен status `Superseded` у ADR. ADR supersession означает замену долговременного архитектурного решения и оформляется отдельным ADR.

### Stale Change

Команда периодически проверяет active changes:

```text
[ ] Есть Owner.
[ ] Why и What Changes всё ещё актуальны.
[ ] Delta не конфликтует с текущей master spec.
[ ] Design соответствует текущей architecture.
[ ] Dependencies и risks актуальны.
[ ] Есть решение: continue / update / supersede / cancel.
```

Stale change нельзя оставлять активным бессрочно: он создаёт ложное представление о планируемом behavior и усложняет новые changes.

### Сохранение Истории

Перед удалением cancelled/superseded active change команда сохраняет достаточную историю в issue, PR или другом принятом decision log:

- Why;
- причина отмены;
- production impact;
- принятые решения;
- ссылка на replacement/recovery;
- важные findings, которые нельзя потерять.

Не создавайте ADR только ради фиксации отменённой задачи. ADR нужен лишь для принятого долговременного архитектурного решения.

### Проверка После Закрытия

Команда вручную проверяет:

```text
[ ] Active change удалён или заменён по repository policy.
[ ] Master spec не получила нереализованную delta.
[ ] Production соответствует master spec либо существует recovery change.
[ ] PR/issues закрыты или связаны с replacement.
[ ] Решение и причина сохранены.
[ ] `openspec validate --all --strict` проходит.
```
