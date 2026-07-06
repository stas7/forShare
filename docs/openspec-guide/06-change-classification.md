# 06. Классификация Change

## Зачем нужна классификация

Не каждое изменение требует одинакового количества OpenSpec artifacts.

Команда должна до начала реализации ответить на четыре вопроса:

```text
1. Меняется ли наблюдаемое поведение?
2. Нужен ли OpenSpec change?
3. Нужен ли подробный design.md?
4. Появляется ли новое долговременное решение для ADR?
```

Классификация нужна не для сокращения документации любой ценой. Она помогает создать ровно те artifacts, которые необходимы для согласования behavior, безопасной реализации и сохранения знаний.

## Основной Decision Flow

```text
Меняется наблюдаемое поведение системы?
│
├─ Нет
│  │
│  ├─ Только formatting, comment, rename или refactoring без изменения behavior
│  │  → OpenSpec change обычно не нужен
│  │
│  └─ Есть риск скрытого изменения contract, configuration или regression behavior
│     → Сначала исследовать; при подтверждении behavior change создать change
│
└─ Да
   → OpenSpec change обязателен
      │
      ├─ Есть существенное technical decision, integration, migration или risk?
      │  → Подробный design.md
      │
      └─ Появляется долговременное architecture decision?
         │
         ├─ Да → ADR
         └─ Нет → `ADR required: no` с обоснованием в design.md
```

Emergency не отменяет decision flow. Оно может изменить порядок и сроки подготовки artifacts, но результаты должны быть восстановлены после стабилизации production.

## Что считается изменением Behavior

OpenSpec change нужен, если меняется ответ на вопрос:

```text
Что система должна делать в этой ситуации?
```

Примеры:

- business rule;
- status или status transition;
- role, permission или ownership rule;
- validation;
- calculation;
- API request/response/error contract;
- message/event contract;
- пользовательское сообщение, если его смысл является частью behavior;
- audit requirement;
- workflow;
- integration behavior;
- behavior при timeout/outage;
- regression behavior, которое должно сохраняться;
- конфигурация приложения, определяющая доступ, route, topic или policy.

Изменение может находиться только в configuration и всё равно быть functional change.

Пример:

```text
Добавить permission к новой role
→ behavior пользователей меняется
→ OpenSpec change нужен
```

## Когда OpenSpec Change обычно не нужен

Change обычно не требуется, если гарантированно не меняется наблюдаемое поведение:

- formatting;
- исправление comment;
- переименование local variable;
- внутренний refactoring с сохранением contracts;
- обновление developer-only documentation;
- перестановка code без изменения execution result;
- техническое исправление test harness без изменения test expectation и строгости assertions.

Решение работать без OpenSpec фиксируется в task или PR:

```text
OpenSpec change не требуется: изменение не влияет на наблюдаемое поведение.
Проверено: API/contracts/configuration/test expectations не изменяются.
```

Если во время реализации обнаружено behavior change, работа останавливается до создания и review OpenSpec change.

## Decision Table

| Тип изменения | OpenSpec Change | Подробный Design | ADR |
|---|---:|---:|---:|
| Новая business capability | Да | Обычно да | Если появляется долговременное решение |
| Изменение существующего business rule | Да | Если есть technical impact/risk | Если меняется architecture rule |
| Bug в behavior, уже описанном master spec | Да | По сложности root cause | Обычно нет |
| Bug в неописанном behavior | Да | По сложности решения | Если появляется новое решение |
| API contract change | Да | Да | Если меняется общий contract pattern |
| Message/schema/topic contract change | Да | Да | Если меняется integration architecture |
| Permission/role mapping change | Да | Если impact не тривиален | Если вводится новый access pattern |
| Application configuration, меняющая behavior | Да | Если есть environment/rollback risk | Обычно нет |
| Database migration | Да, если влияет на behavior/data contract | Да | Если меняется data architecture |
| Internal refactoring без behavior change | Обычно нет | Может быть отдельный technical plan | Если вводится долговременный pattern |
| Formatting/comment/local rename | Нет | Нет | Нет |
| Emergency functional fix | Да, допускается ускоренный порядок | Минимум risk/rollback до release | После стабилизации, если требуется |

`Обычно` и `если` требуют инженерного решения, а не автоматического пропуска artifact.

## Когда Design обязателен

Подробный `design.md` нужен, если change затрагивает хотя бы одну существенную область:

- API или message contract;
- authentication, authorization или security;
- database schema, migration или backfill;
- external/platform integration;
- asynchronous processing;
- concurrency или transaction boundary;
- idempotency, retry или deduplication;
- data retention или sensitive data;
- multi-component flow;
- backward compatibility;
- feature flag, controlled rollout или rollback;
- observability и operational risk;
- крупный refactoring;
- несколько реалистичных технических alternatives.

Для небольшого change design всё равно должен содержать решение по применимости:

```text
Подробный design не требуется: change использует существующий mapping pattern,
не меняет architecture, data, integration или rollout.
```

В archived bugfix example краткий design сохранён даже без ADR, потому что он объясняет mapping и подтверждает отсутствие нового architecture decision.

## Когда ADR обязателен

ADR нужен, если change создаёт или меняет решение, которое:

- влияет на будущие changes;
- задаёт границу modules или layers;
- определяет общий API/message/error pattern;
- вводит обязательный service или transition mechanism;
- выбирает или запрещает dependency;
- задаёт общий security/data/integration constraint;
- будет проверяться в будущих PR;
- потребует объяснения через несколько месяцев;
- заменяет existing architecture decision.

Примеры:

```text
Все status transitions проходят через RequestStatusTransitionService.
→ ADR нужен

Для одного error case добавляется новый code в существующий mapper.
→ ADR не нужен

Audit event отправляется через уже принятый platform audit standard.
→ Новый ADR приложения обычно не нужен; ссылка на existing standard уместна
```

## Когда ADR не нужен

ADR обычно не создаётся для:

- локального bugfix;
- отдельного endpoint, использующего existing pattern;
- configuration value;
- task breakdown;
- temporary implementation detail;
- решения, полностью определённого existing ADR/platform standard;
- выбора имени class или method;
- небольшого refactoring без нового общего constraint.

В `design.md` всё равно фиксируется:

```text
ADR required: no
Reason: <почему новое долговременное решение не появилось>
Existing decision/standard: <ссылка, если применимо>
```

Отсутствие ADR является результатом Design Harvest, а не отсутствием анализа.

## Platform Functionality

Использование платформы само по себе не определяет необходимость ADR.

Нужно разделить:

```text
Платформа предоставляет mechanism.
Приложение определяет business behavior и configuration.
```

Пример audit:

```text
Platform responsibility:
- outbox;
- Kafka producer;
- retry;
- DLQ;
- technical metrics.

Application responsibility:
- обязательность audit event;
- event type;
- payload;
- topic mapping;
- schema version;
- запрет чувствительных данных.
```

Если приложение использует existing platform standard, новый ADR обычно не нужен. Если приложение меняет общий способ audit delivery или вводит новый обязательный pattern, ADR нужен.

## Configuration Changes

Configuration рассматривается как часть implementation.

OpenSpec change нужен, если configuration определяет:

- кому доступна функция;
- какое business rule включено;
- куда отправляется contract message;
- какой workflow выполняется;
- какой provider используется;
- какое observable fallback behavior применяется.

OpenSpec change может не требоваться для технического environment value, если behavior и contract не меняются.

Пример:

```text
Изменить broker hostname без изменения topic/contract/behavior
→ обычно infrastructure change без product OpenSpec

Изменить topic, на который подписан внешний consumer
→ contract change, OpenSpec обязателен
```

## Emergency Change

Production incident допускает ускоренный порядок, но не отменяет фиксацию behavior и risk.

### Минимум до Release

Команда вручную фиксирует:

- incident и impact;
- подтверждённое текущее поведение;
- ожидаемое безопасное поведение;
- минимальный scope;
- risk;
- rollback;
- critical Scenario;
- owner решения;
- план восстановления OpenSpec artifacts.

### После Стабилизации

Команда:

1. Завершает proposal с `Why` и `What Changes`.
2. Добавляет или уточняет delta spec.
3. Восстанавливает фактический design.
4. Добавляет tests и regression evidence.
5. Выполняет Design Harvest.
6. Создаёт/обновляет ADR, если появилось архитектурное решение.
7. Выполняет обычный archive и post-archive validation.

В change/PR отмечается, какие artifacts оформлены после release и почему.

## Как фиксировать Exception

Если команда отклоняется от стандартного процесса, она записывает:

```text
Exception:
Что пропускается или откладывается.

Reason:
Почему стандартный порядок невозможно выполнить.

Risk:
Какой риск создаёт exception.

Owner:
Кто отвечает за восстановление процесса.

Deadline / Trigger:
Когда или после какого события exception закрывается.
```

Фраза «задача маленькая» не является достаточным обоснованием.

## Примеры Классификации

### Новая витрина продуктов

```text
Behavior change: да
OpenSpec change: да
Design: да — access, configuration, platform integrations
ADR: возможно — если появляется новый общий ProductAvailabilityService
```

### Исправление сообщения после начала согласования

```text
Behavior change: да — error contract и пользовательское сообщение
OpenSpec change: да
Design: краткий — existing mapper/localization pattern
ADR: нет — новое architecture decision не появилось
```

### Переименование Local Variable

```text
Behavior change: нет
OpenSpec change: нет
Design: нет
ADR: нет
```

### Новый Kafka Topic для Внешнего Consumer

```text
Behavior/contract change: да
OpenSpec change: да
Design: да — schema, delivery, compatibility, rollout
ADR: если меняется общий integration pattern
```

## Ответственность Ролей

- Analyst подтверждает, меняется ли ожидаемое поведение.
- Developer оценивает technical impact и необходимость design.
- QA оценивает regression/testability impact.
- Reviewer независимо проверяет классификацию.
- Решение по ADR принимается в design/review и подтверждается при Design Harvest.

Классификацию нельзя оставлять только на авторе code.

## Результат Классификации

До implementation команда должна иметь явный результат:

```text
OpenSpec change required: yes / no
Reason: <обоснование>

Detailed design required: yes / no
Reason: <обоснование>

ADR required: yes / no / decide at Design Harvest
Reason or existing decision: <обоснование/ссылка>

Emergency exception: yes / no
Owner and recovery plan: <если применимо>
```

Этот результат фиксируется в task, proposal или design и проверяется Reviewer.
