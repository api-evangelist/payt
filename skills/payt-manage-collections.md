---
name: manage-collections
description: Drive the Payt collections workflow - pause/resume invoice follow-up, message debtors, and work collection tasks.
api: Payt Customer API v1
base_url: https://api.paytsoftware.com
operations:
  - getV1Invoices
  - patchV1InvoicesBlock
  - patchV1InvoicesResume
  - postV1Messages
  - getV1Tasks
  - patchV1TasksIdMarkCompleted
method: generated
source: openapi/payt-openapi-original.json
---

# Manage Payt collections

Control the automated dunning flow for specific invoices and follow up with debtors.

## Prerequisites

- An OAuth 2.0 Bearer access token with the write scopes for invoices, messages and
  tasks (e.g. `invoices:write`). Send as `Authorization: Bearer <token>`.
- The target `administration_id`.

## Block / resume invoice follow-up

1. **Find the invoices** — `getV1Invoices` (`GET /v1/invoices?administration_id=<id>`),
   paging with `cursor`.
2. **Block** the follow-up flow — `patchV1InvoicesBlock` (`PATCH /v1/invoices/block`)
   with the invoice identifiers. This pauses reminders for those invoices.
3. **Resume** when resolved — `patchV1InvoicesResume` (`PATCH /v1/invoices/resume`).
   Only invoices blocked through the API can be resumed through the API.

## Message a debtor

- `postV1Messages` (`POST /v1/messages`) to send a message to a debtor. The endpoint
  accepts multiple records; inspect the `{success, count, errors, warnings}` result —
  `success:true` can still list per-record `errors`.

## Work tasks

1. **List tasks** — `getV1Tasks` (`GET /v1/tasks?administration_id=<id>`).
2. **Complete a task** — `patchV1TasksIdMarkCompleted`
   (`PATCH /v1/tasks/{id}/mark_completed`); reverse with `patchV1TasksIdUnmarkCompleted`.

## Guardrails

- A `403 {"code":"forbidden","message":"Missing permission: <scope>"}` means the token
  lacks the required scope — prompt the user to grant it.
- Rate limit 10 req/s per token; back off on `429`.
- Pair with webhooks (`webhooks/payt-webhooks.yml`): react to `invoice_paid`,
  `invoice_blocked`, `invoice_resumed` and `case_created` rather than polling.
