# Инструкции для AI-агентов

## Защита test expectations

Если новый или изменённый production-код не проходит существующий test, считайте это конфликтом реализации с зафиксированным поведением.

До явного решения человека не изменяйте, не удаляйте, не пропускайте и не ослабляйте test, assertions, fixtures, mocks, golden files или coverage thresholds только ради успешного прохождения нового кода.

При конфликте:

1. Остановите автоматическое исправление.
2. Покажите failing test, actual и ожидаемое поведение.
3. Определите вероятную причину: defect production-код, изменение requirement или defect test.
4. Предложите варианты и объясните влияние на OpenSpec и совместимость.
5. Дождитесь явного решения человека перед изменением test expectation.

Если меняется поведение, сначала обновите active OpenSpec change и delta spec.

## OpenSpec

- Перед functional change прочитайте project context, затронутые master specs и active change.
- Future behavior описывайте через `openspec/changes/<change-name>/`, а не прямым изменением master spec.
- Не переписывайте semantic content archived change; новое behavior оформляйте новым change.
- Поддерживайте согласованность `proposal → delta spec → design → tasks → code → tests`.
- Инструкции для OpenSpec artifacts находятся в `openspec/AGENTS.md`; не дублируйте их здесь.

## ADR

Долговременные архитектурные решения сохраняйте в ADR.

- Используйте последовательную нумерацию.
- Добавляйте ссылки на archived change и `design.md`.
- Не переписывайте принятое решение задним числом.
- При замене создайте новый ADR, старому назначьте `Superseded`.
- Свяжите каждый archived change с ADR или отметьте его как `non-architectural`.

## Проверки проекта

Для этого documentation example используются следующие проверки:

```text
OpenSpec validation после archive: `openspec validate --all --strict`
Static analysis: не запускается здесь — implementation code находится вне documentation example
Automated tests: не запускаются здесь — PR содержит моделируемое evidence реального проекта
Manual QA: не выполняется здесь — результаты моделируются в pull-request.md
```

## Локальные инструкции

Более локальный `AGENTS.md` может уточнять эти правила, но не должен ослаблять защиту test expectations и обязательный OpenSpec process.
