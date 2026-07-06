# Proposal: fix-withdrawal-error-message

## Why

Когда автор пытается отозвать заявку после начала согласования, API возвращает общий error code `REQUEST_STATUS_NOT_WITHDRAWABLE`.

UI не может отличить начало согласования от других запрещённых статусов и показывает сообщение «Операция недоступна». Пользователь не понимает, почему отзыв невозможен, и обращается в поддержку.

Нужно возвращать отдельный стабильный error code и понятное пользовательское сообщение для уже начавшегося согласования.

## What Changes

- Для попытки отзыва после начала согласования API возвращает `409` и error code `APPROVAL_ALREADY_STARTED`.
- UI отображает сообщение «Заявку нельзя отозвать: согласование уже началось».
- Статус и данные заявки не изменяются.
- Существующее правило возможности отзыва не меняется.

## Out of Scope

- Изменение status transition rules.
- Возможность отзыва после начала согласования.
- Изменение permission или role mappings.
- Новый audit event.
- Изменение общего API error envelope.

## Affected Areas

- Withdrawal domain error mapping.
- Request API error response.
- UI message catalog.
- API и UI tests.

## Risks and Constraints

- Existing consumers могут использовать общий error code.
- Новый code должен добавляться в существующий API error catalog.
- Нельзя менять HTTP status для других причин запрещённого отзыва.

## Success Criteria

- Approval started возвращает `409 / APPROVAL_ALREADY_STARTED`.
- UI показывает установленное сообщение.
- Заявка не изменяется.
- Другие withdrawal errors сохраняют существующие codes.
- Regression withdrawal и approval flows пройдена.
