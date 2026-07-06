# ADR Traceability

Этот index связывает archived changes с долговременными архитектурными решениями.

Каждый archived change должен:

- ссылаться на один или несколько ADR;
- либо быть явно классифицирован как `Non-architectural`.

## Traceability Table

| Archived Change | ADR | ADR Status | Классификация | Обоснование |
|---|---|---|---|---|
| `<YYYY-MM-DD-change-name>` | `<ADR-NNN>` | `Accepted` | `Architectural` | `<какое долговременное решение появилось>` |
| `<YYYY-MM-DD-change-name>` | — | — | `Non-architectural` | `<почему нового архитектурного решения нет>` |

Если один change связан с несколькими ADR, используйте отдельную строку для каждого ADR или перечислите их в одной ячейке по принятому правилу repository.

## Classification Rules

### Architectural

Используйте `Architectural`, если change:

- создаёт долговременное architecture decision;
- изменяет existing decision;
- вводит общий pattern или constraint;
- влияет на будущие changes/PR reviews;
- требует нового ADR или обновления existing ADR.

### Non-Architectural

Используйте `Non-architectural`, если change:

- не создаёт нового долговременного решения;
- использует existing ADR/platform standard;
- является локальным bugfix;
- меняет mapping/configuration в рамках существующего pattern;
- содержит только implementation details конкретного change.

`Non-architectural` требует краткого обоснования. Пустая ADR cell без classification считается незавершённой traceability.

## ADR Source Links

Каждый ADR должен содержать section `Источники`:

```text
Archived change: <path>
Design: <path-to-archived-design.md>
Supersedes: <ADR-NNN или не применимо>
Superseded by: <ADR-NNN или не применимо>
```

После archive links должны указывать на dated archived path, а не на удалённый active change.

## Superseded Decision

Если новое решение заменяет existing ADR:

1. Создайте новый ADR.
2. Укажите в новом ADR `Supersedes: ADR-NNN`.
3. Измените status старого ADR на `Superseded`.
4. Укажите в старом ADR `Superseded by: ADR-MMM`.
5. Обновите traceability rows обоих решений.
6. Не переписывайте старый Decision задним числом.

Пример:

| Archived Change | ADR | ADR Status | Классификация | Обоснование |
|---|---|---|---|---|
| `2026-07-06-add-request-withdrawal` | `ADR-001` | `Superseded` | `Architectural` | Первоначальное правило status transitions |
| `2026-10-12-replace-status-workflow` | `ADR-004` | `Accepted` | `Architectural` | Новое решение заменяет ADR-001 |

## Active Change до Archive

Если ADR принят до archive, временно допустима запись:

| Change | ADR | ADR Status | Классификация | Обоснование |
|---|---|---|---|---|
| `<active-change-name>` | `<ADR-NNN>` | `Accepted; archive pending` | `Architectural` | `<decision>` |

После archive active name заменяется на dated archived change, а source links ADR обновляются.

## Review Checklist

```text
[ ] Каждый archived change присутствует в index.
[ ] Для Architectural change указан ADR.
[ ] Для Non-architectural change есть обоснование.
[ ] ADR status совпадает с самим ADR.
[ ] Source links указывают на archived paths.
[ ] Superseded/Superseded by links симметричны.
[ ] Номера ADR не переиспользуются.
[ ] Старые Decisions не переписаны задним числом.
```
