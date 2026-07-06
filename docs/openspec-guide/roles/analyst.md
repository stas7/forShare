# Роль: Analyst

## Главная ответственность

Analyst отвечает за смысл change и однозначное ожидаемое поведение.

Роль Analyst может выполнять один или несколько участников команды. Важно, чтобы для каждого change был определён owner поведения и не оставалось требований, за которые никто не отвечает.

## За что отвечает Analyst

- бизнес-проблема и цель change;
- `Why` и `What Changes` в proposal;
- scope и out of scope;
- requirements и Scenarios;
- согласование ожидаемое поведение;
- связь delta spec с master spec;
- разрешение неоднозначностей;
- business acceptance criteria;
- смысл наблюдаемое поведение после archive.

## Explore

До создания artifacts Analyst:

- выясняет, кто пользователь или system actor;
- формулирует проблему и ожидаемый результат;
- отделяет текущий факт от предположения;
- определяет заинтересованные стороны;
- выявляет бизнес-правила, ограничения и unknowns;
- для brownfield отделяет текущее поведение от ожидаемое поведение;
- для bugfix сопоставляет фактическое поведение с master spec.

Код может подтвердить текущее поведение, но не является автоматическим доказательством правильного requirement.

## Proposal

Analyst является owner business content `proposal.md`.

Он отвечает за:

```text
Why
Почему change нужен, какую проблему решает и какую ценность создаёт.

What Changes
Какое поведение или capability изменится.

Out of Scope
Что намеренно не входит в change.

Success Criteria
Как команда подтвердит достижение цели.
```

Analyst привлекает Developer для technical impact, QA для проверяемости и Reviewer для независимой проверки scope.

## Specification

Analyst является owner requirements и Scenarios.

Он проверяет:

- requirement описывает наблюдаемое поведение;
- используется SHALL-утверждение;
- preconditions сформулированы явно;
- action и expected result однозначны;
- negative и поведение при ошибке определены;
- указано, что не должно измениться при отказе;
- роли, permissions и ownership rules не смешаны;
- тексты после Requirement, Scenario, GIVEN, WHEN, THEN и AND написаны на русском языке;
- детали реализации не попали в specification без contract reason.

## Master Spec и Delta

До написания delta Analyst читает затронутую master spec и связанные capabilities.

Он определяет:

- является Requirement новым, изменяемым или удаляемым;
- нет ли conflicts с существующими requirements;
- какое regression behavior нужно сохранить;
- какое поведение должно стать частью master spec после archive.

Future behavior описывается в delta active change. Master spec не меняется вручную заранее.

## Platform Boundary

Analyst описывает бизнес-поведение, а не внутреннее устройство корпоративной платформы.

Он должен понимать разницу:

```text
Платформа предоставляет access control mechanism.
Приложение определяет permission и бизнес-правило доступа.

Платформа доставляет audit message.
Приложение определяет обязательность business event и его business fields.
```

Topic, schema version или API path включаются в spec только если являются внешним contract или обязательным наблюдаемое поведение. Producer, outbox table и retry implementation относятся к design.

## Spec Review

На review Analyst:

- объясняет Why и ожидаемое поведение;
- отвечает на вопросы по scope;
- закрывает неоднозначности;
- принимает или отклоняет предложенные edge cases;
- подтверждает, что Scenarios отражают business intent;
- даёт go/no-go по смысловой готовности.

Analyst не должен соглашаться с расплывчатой формулировкой только ради начала implementation.

## Во время Implementation

Analyst:

- оперативно отвечает на вопросы по поведению;
- не согласует изменение requirement только в PR comment или чате;
- обновляет delta spec до изменения implementation/test expectation;
- контролирует scope creep;
- фиксирует новые business decisions в OpenSpec artifacts.

Если code невозможно реализовать по текущему requirement, это повод пересмотреть requirement явно, а не позволить implementation определить поведение молча.

## Test Expectation Conflicts

Если новый code конфликтует с существующим test expectation, Analyst отвечает за решение по ожидаемое поведение.

Он рассматривает:

- соответствует ли test master spec;
- изменяет ли active change поведение намеренно;
- является ли конфликт defect production-код;
- является ли test устаревшим или ошибочным;
- как решение влияет на compatibility и regression.

Test expectation изменяется после явного решения и синхронизации delta spec, если меняется behavior.

## QA и Acceptance

Analyst:

- помогает разрешать спорные QA results;
- подтверждает business meaning ошибок и сообщений;
- проверяет, что реализованный результат соответствует Success Criteria;
- не подменяет QA sign-off;
- участвует в решении о расширении scope при новых findings.

## Design Harvest

Перед archive Analyst:

- проверяет финальное наблюдаемое поведение;
- убеждается, что delta spec соответствует реализации;
- подтверждает, что master spec после archive будет нормативной;
- определяет, влияет ли архитектурное решение на business constraints;
- проверяет, что детали реализации не переносятся в master spec;
- участвует в подготовке ADR, если решение требует business context.

## После Archive

Analyst вручную проверяет:

- master spec содержит финальное ожидаемое поведение;
- `Purpose` capability заполнен содержательно;
- Scenarios не потерялись при применении delta;
- archived change сохраняет Why и историю решения;
- ссылки на ADR помогают понять ограничение, но не превращают master spec в architecture document.

## Что Analyst не должен делать

Analyst не должен:

- описывать class/method call как requirement;
- выбирать implementation вместо Developer;
- переносить весь design в master spec;
- считать UI visibility достаточным access control;
- менять test expectation без анализа actual/expected;
- расширять scope без обновления proposal;
- оставлять unknowns внутри финальной нормативной формулировки.

Плохо:

```md
Система SHALL вызывать `RequestService.validateWithdrawal()`.
```

Хорошо:

```md
Система SHALL отклонять отзыв заявки после начала процесса согласования.
```

## Результат работы роли

После работы Analyst команда понимает:

- зачем выполняется change;
- что именно изменяется;
- как система должна вести себя;
- что остаётся без изменений;
- как проверить результат;
- какое поведение попадёт в master spec.
