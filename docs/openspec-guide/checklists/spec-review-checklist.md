# Checklist: Spec Review

Spec Review проводится до основной implementation.

Цель review — подтвердить, что команда одинаково понимает Why, What Changes и наблюдаемое поведение, а delta spec корректно изменяет master spec.

## 1. Proposal

```text
[ ] proposal.md содержит обязательный раздел Why.
[ ] Why объясняет проблему или возможность, цель и ценность change.
[ ] proposal.md содержит обязательный раздел What Changes.
[ ] What Changes описывает scope изменения поведения или возможностей.
[ ] Out of Scope описан явно.
[ ] Affected Areas перечислены.
[ ] Risks and Constraints описаны.
[ ] Success Criteria проверяемы.
[ ] Proposal не смешивает несколько независимых changes.
```

## 2. Master Spec и Delta

```text
[ ] Затронутая master spec прочитана до написания delta.
[ ] Проверены связанные capabilities и возможные противоречия.
[ ] Для каждого изменения правильно выбрана операция ADDED / MODIFIED / REMOVED.
[ ] ADDED используется только для нового Requirement.
[ ] MODIFIED содержит полную новую версию изменяемого Requirement, а не отдельный фрагмент.
[ ] REMOVED явно указывает удаляемое поведение и причину.
[ ] Delta не изменяет поведение за пределами scope proposal.
[ ] Сохраняемое поведение не удалено случайно.
[ ] Для brownfield текущее поведение подтверждён или unknowns обозначены явно.
[ ] Для bugfix master spec сопоставлена с ожидаемое поведение.
```

Master spec не изменяется вручную для описания future behavior до archive. Источником будущего изменения является delta spec active change.

## 3. OpenSpec Syntax

```text
[ ] Каждый Requirement имеет заголовок `### Requirement: ...`.
[ ] Каждый Requirement содержит нормативное SHALL-утверждение.
[ ] Каждый Requirement имеет минимум один `#### Scenario: ...`.
[ ] Scenarios используют GIVEN, WHEN, THEN и при необходимости AND.
[ ] Текст после Requirement, Scenario, GIVEN, WHEN, THEN и AND написан на русском языке.
[ ] Requirement и Scenario имеют уникальные понятные названия в пределах capability.
[ ] Markdown structure соответствует формату OpenSpec.
```

Ключевые слова OpenSpec и общепринятые technical terms сохраняются. Описательный текст пишется на русском языке.

## 4. Observable Behavior

```text
[ ] Specification отвечает на вопрос «что должна делать система».
[ ] Specification не описывает классы, методы, файлы или внутренние layers.
[ ] Preconditions сформулированы явно.
[ ] User/system action сформулирован однозначно.
[ ] Expected result можно проверить.
[ ] Указано, что не должно измениться при отклонённой операции.
[ ] Error behavior описано через наблюдаемый result или contract.
[ ] Нет слов «корректно», «правильно», «быстро» без измеримого значения.
[ ] Нет скрытых предположений, известных только автору requirement.
```

Implementation details относятся к `design.md`.

API path, error code, event type, schema version или topic могут находиться в spec, только если они являются внешним contract или обязательным наблюдаемое поведение. Выбор library, producer, outbox table, retry implementation и имена внутренних classes относятся к design.

## 5. Scenarios

```text
[ ] Happy path описан.
[ ] Negative cases описаны.
[ ] Authorization failure описан, если применимо.
[ ] Validation errors описаны, если применимо.
[ ] Boundary values описаны, если применимо.
[ ] Duplicate/repeated action описана, если применимо.
[ ] Concurrency behavior описано, если оно наблюдаемо и существенно.
[ ] Integration unavailable behavior описано, если применимо.
[ ] Empty result / no data behavior описано, если применимо.
[ ] Backward compatibility behavior описано, если применимо.
[ ] Regression-critical behavior сохранено отдельным Scenario, если необходимо.
```

Не добавляйте искусственные Scenarios для неприменимых случаев. Если риск существенный, но полностью технический, перенесите его в design и test strategy.

## 6. Roles, Permissions and Access

Если change затрагивает access control:

```text
[ ] Пользователь или system actor указан.
[ ] Необходимый permission или business role определён.
[ ] Ownership rule описан отдельно от platform permission, если применимо.
[ ] Поведение пользователя без доступа описано.
[ ] Specification не полагается только на скрытие action в UI.
[ ] Не раскрываются данные или существование resource без разрешения.
```

## 7. Audit, Events and Integrations

Если change затрагивает audit, messaging или integration:

```text
[ ] Указано, какое business event или observable integration result требуется.
[ ] Определён момент, когда event считается созданным с точки зрения бизнес-поведение.
[ ] Обязательные business fields перечислены, если они являются contract.
[ ] Sensitive data и запрещённые fields обозначены, если применимо.
[ ] Поведение при недоступности integration описано на observable уровне.
[ ] Platform implementation mechanics не скопированы в specification.
[ ] Topic/schema/API details включены только при наличии contract reason.
```

## 8. Проверяемость

```text
[ ] Каждый Scenario можно проверить автоматизированно или вручную.
[ ] Для каждого THEN понятен источник фактического результата.
[ ] Необходимые test data можно подготовить.
[ ] Необходимые roles, permissions и configuration доступны в test environment.
[ ] QA понимает основной test plan.
[ ] Regression scope определён.
[ ] Непроверяемые формулировки переписаны.
```

## 9. Review Roles

```text
[ ] Аналитик подтвердил ожидаемое поведение и scope.
[ ] Разработчик 1 подтвердил реализуемость и отсутствие неизвестных blockers.
[ ] Разработчик 2 проверил conflicts, architecture constraints и contracts.
[ ] Тестировщик 1 подтвердил testability и acceptance plan.
[ ] Тестировщик 2 проверил edge cases и regression risks.
```

## 10. Validation и Go / No-Go

Команда запускает:

```text
openspec validate <change-name> --strict
```

После выполнения:

```text
[ ] Validation завершилась успешно.
[ ] Warnings разобраны, а не проигнорированы.
[ ] Review comments закрыты.
[ ] Unknowns закрыты или явно блокируют implementation.
[ ] Команда приняла явное решение go/no-go.
```

Implementation начинается только после `go`.
