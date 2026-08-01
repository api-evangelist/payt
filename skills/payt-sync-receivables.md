---
name: sync-receivables
description: Incrementally sync Payt administrations, debtors, invoices and payments into an external system (ERP/accounting/data warehouse).
api: Payt Customer API v1
base_url: https://api.paytsoftware.com
operations:
  - getV1Administrations
  - getV1Debtors
  - getV1Invoices
  - getV1Payments
method: generated
source: openapi/payt-openapi-original.json
---

# Sync receivables from Payt

Keep an external system in step with Payt's invoices, debtors and payments.

## Prerequisites

- An OAuth 2.0 Bearer access token with `debtors:read` and `invoices:read`
  (add `payments` read for payments). Send it as `Authorization: Bearer <token>`.
- The `administration_id` you are syncing. Get the list first if unknown.

## Steps

1. **List administrations** — call `getV1Administrations` (`GET /v1/administrations`)
   to enumerate the tenants you can access. Each carries a `last_successful_import_at`.
2. **Page through debtors** — call `getV1Debtors` (`GET /v1/debtors?administration_id=<id>`).
   Follow pagination: read `pagination.cursor` from the response and pass it back as
   `cursor` until no cursor is returned.
3. **Page through invoices** — call `getV1Invoices`
   (`GET /v1/invoices?administration_id=<id>`) the same way. Request only the fields you
   need via the `fields` parameter.
4. **Page through payments** — call `getV1Payments` for reconciliation.
5. **Persist Payt ids** — store each record's numeric `id` and link it to your own
   key. Do NOT match on `invoice_number` alone: it is unique only within one
   administration.

## Incremental sync

- Add `updated_after=<ISO8601 timestamp>` to request only records changed since the
  last run.
- After processing the last page, store the greatest `updated_at` seen and use it as
  the next `updated_after`.
- If a page returns no records, **reuse the previous timestamp** — do not advance it,
  or you may skip records.

## Conventions & guardrails

- Monetary values are strings with two decimals; timestamps are UTC ISO 8601.
- Rate limit: 10 requests/second per token. On `429`, back off exponentially.
- Prefer index endpoints over firing many show requests.
- See `conventions/payt-conventions.yml` and `errors/payt-problem-types.yml`.
