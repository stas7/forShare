# Example: fix-withdrawal-error-message

Компактный пример завершённого bugfix change без нового ADR.

## Задача

После начала согласования API возвращал общий error code, а UI показывал непонятное сообщение.

Change добавляет отдельный observable error contract:

```text
HTTP 409
error.code = APPROVAL_ALREADY_STARTED
message = Заявку нельзя отозвать: согласование уже началось
```

Business rule и architecture не меняются.

## Почему OpenSpec Change нужен

Хотя исправление небольшое, оно меняет:

- API error contract;
- пользовательское сообщение;
- regression behavior.

Поэтому change оформлен через proposal, delta spec, design, tasks и archive.

## Почему ADR не нужен

Design содержит явное решение:

```text
ADR required: no
```

Причина:

- используется существующая domain error;
- используется существующий API error envelope;
- используется существующий localization pattern;
- новый architecture pattern или constraint не создаётся.

Технический mapping важен для истории конкретного bugfix, но не является долговременным архитектурным решением. Он остаётся в archived `design.md` и tests.

## Design Harvest

Перед archive команда проверила:

- реализация соответствует delta spec;
- общий error mapping не изменился для других случаев;
- новых архитектурное решениеs нет;
- ADR создавать не нужно;
- change должен быть отмечен `Non-architectural`.

## Что сделал OpenSpec автоматически

После ручного запуска:

```text
openspec validate fix-withdrawal-error-message --strict
openspec archive fix-withdrawal-error-message --yes
```

OpenSpec CLI автоматически:

1. Применил ADDED Requirement к master spec `approval-requests`.
2. Переместил active change в `changes/archive/2026-07-06-fix-withdrawal-error-message/`.

## Что команда сделала вручную

- согласовала expected поведение при ошибке;
- выполнила implementation, tests, QA и regression;
- выполнила Design Harvest;
- приняла решение `ADR required: no`;
- добавила change в traceability index как `Non-architectural`;
- проверила master spec и archived change;
- запустила post-archive validation.

## Структура после Archive

```text
fix-withdrawal-error-message/
├── README.md
├── docs/
│   └── adr/
│       └── README.md
└── openspec/
    ├── specs/
    │   └── approval-requests/
    │       └── spec.md
    └── changes/
        └── archive/
            └── 2026-07-06-fix-withdrawal-error-message/
                ├── proposal.md
                ├── design.md
                ├── tasks.md
                └── specs/
                    └── approval-requests/
                        └── spec.md
```

## Как читать пример

```text
archived proposal.md
→ archived delta spec
→ archived design.md
→ archived tasks.md
→ master spec
→ ADR traceability index
```

Обратите внимание: отсутствие ADR не означает отсутствие design или Design Harvest. Команда всё равно обязана проанализировать решение и явно обосновать классификацию `Non-architectural`.
