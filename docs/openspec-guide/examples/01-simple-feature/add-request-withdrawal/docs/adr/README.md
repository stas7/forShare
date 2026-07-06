# ADR Traceability

| Change | ADR | Status | Классификация |
|---|---|---|---|
| `2026-07-06-add-request-withdrawal` | `ADR-001` | Accepted | Architectural |

## Почему один ADR

Change содержит два заметных технических аспекта:

- единый `RequestStatusTransitionService`;
- публикация audit event через platform audit module и Kafka.

ADR создан только для первого аспекта, потому что он вводит долговременное архитектурное правило приложения.

Для platform audit отдельный ADR не нужен: приложение следует существующему corporate standard. Topic mapping, payload и schema сохраняются в design и конфигурация приложения.

## Правила

- Каждый archived change добавляется в таблицу.
- До archive допустима запись `archive pending`, если ADR уже принят.
- Если change не создаёт долговременного архитектурного решения, в ADR указывается `—`, а классификация — `Non-architectural`.
- Если решение заменено, создаётся новый ADR, а старый получает status `Superseded`.
- Номера ADR не переиспользуются.
- После archive source links ADR обновляются на archived path.
