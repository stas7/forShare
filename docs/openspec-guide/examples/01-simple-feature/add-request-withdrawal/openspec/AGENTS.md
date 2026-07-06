# Инструкции для OpenSpec artifacts

## Перед изменением кода

1. Прочитайте project context и затронутые master specs.
2. Для functional change создайте `openspec/changes/<verb-noun>/`.
3. Не изменяйте master spec для описания future behavior до archive.
4. Проверьте связанные contracts, модели и фактическую реализацию.

## Структура change

- `proposal.md` — intent, scope, non-goals и impact.
- `specs/<capability>/spec.md` — `ADDED`, `MODIFIED`, `REMOVED` requirements.
- `design.md` — technical approach, migrations, risks и rollback.
- `tasks.md` — небольшие нумерованные задачи с проверками.

## Specification rules

- Requirement имеет заголовок `### Requirement: ...` и SHALL-утверждение.
- Каждый Requirement имеет минимум один `#### Scenario: ...`.
- Scenarios используют `GIVEN`, `WHEN`, `THEN`, `AND`.
- Текст после Requirement, Scenario, GIVEN, WHEN, THEN и AND пишется на языке проекта.
- `spec.md` описывает наблюдаемое поведение.
- Классы, библиотеки, файлы и детали реализации относятся к `design.md`.

## Implementation rules

- Не меняйте test expectation автоматически при конфликте с production-код.
- Зафиксируйте actual и ожидаемое поведение и запросите решение человека.
- При изменении поведения сначала обновите active change и delta spec.
- Не изменяйте несвязанные участки и сохраняйте пользовательские изменения.
- Не изменяйте semantic content archived change задним числом; используйте новый change или новый ADR.

## Минимальная проверка

```text
OpenSpec validation после archive: `openspec validate --all --strict`
Static analysis: не запускается здесь — implementation code находится вне documentation example
Automated tests: не запускаются здесь — PR содержит моделируемое evidence реального проекта
Manual QA: не выполняется здесь — результаты моделируются в pull-request.md
```
