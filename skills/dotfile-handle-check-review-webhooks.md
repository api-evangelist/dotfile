---
name: Consume Dotfile webhooks and drive check review
description: Register a webhook endpoint, react to the 46 Dotfile events (above all Check.ReviewNeeded), reconcile from the webhook log when a delivery is missed, and record verdicts back over the API.
api: openapi/_original/dotfile-openapi.json
generated: '2026-08-17'
method: generated
source: https://docs.dotfile.com/reference/webhooks-guide
operations:
  - webhook-get-many
  - webhook-create-one
  - webhook-get-one
  - webhook-update-one
  - webhook-delete-one
  - webhook-log-get-many
  - check-get-many
  - case-get-one
  - aml-review-hits
  - aml-review
  - document-review
  - document-force-review
  - id-document-review
  - id-verification-review
  - ekyc-review
  - electronic-signature-review
  - fraud-database-review
  - online-reputation-review
---

# Consume Dotfile webhooks and drive check review

Dotfile checks run asynchronously, so webhooks are the primary result channel. There are **46
subscribable events** across nine families: `Case` (17), `CaseReport` (1), `Note` (3),
`NoteComment` (3), `Individual` (6), `Company` (6), `Check` (6), `DocumentOrder` (2),
`AutonomyChatRun` (2). Full catalog with descriptions: `asyncapi/dotfile-webhooks.yml`.

## 1 — Register the endpoint

`webhook-create-one` (`POST /v1/webhooks`) with your HTTPS endpoint and an `events[]` array.
Dotfile then POSTs a raw JSON body to it. A workspace holds up to **50 webhooks** — the 51st
returns `400` "Maximum items reached (default: 50)".

Manage with `webhook-get-many`, `webhook-get-one`, `webhook-update-one`, `webhook-delete-one`.

A minimal, useful subscription:

```
Check.Started, Check.ReviewNeeded, Check.Approved, Check.Rejected, Check.Expired,
Case.StatusUpdated, Case.RiskUpdated, Case.ReviewConfirmed, Case.ReviewDue,
DocumentOrder.Completed, DocumentOrder.Failed, CaseReport.Generated
```

## 2 — Verify the delivery yourself

**Dotfile publishes no webhook signing secret, no HMAC header, and no replay guard.** Nothing in the
payload proves it came from Dotfile. Treat a delivery as an untrusted *hint*:

1. Accept it, return `2xx` fast, enqueue.
2. Re-read the authoritative state over the API — `case-get-one` (accepts your `external_id`) or
   `check-get-many` filtered to the case — before taking any consequential action.
3. Harden the endpoint by other means: an unguessable path segment, a source-IP allowlist, mutual
   TLS at your edge.

Never approve, reject, disburse or provision off the webhook body alone.

## 3 — Read the sub_event, not just the event

`Case.Updated`, `Individual.Updated` and `Company.Updated` are umbrella events. A `sub_event` is
**always** present and is the field that says what actually changed:

- `Case.*` — `StatusUpdated`, `FlagsUpdated`, `ContactHasActionsUpdated`,
  `ReviewerHasActionsUpdated`, `InfoUpdated`, `TemplateUpdated`, `RiskUpdated`, `MetadataUpdated`,
  `TagsUpdated`, `AssigneeUpdated`
- `Individual.*` / `Company.*` — `InfoUpdated`, `MarkedAsRelevant`, `MarkedAsNotRelevant`

Switching on `event` alone will make every property change look identical. Two traps:
`Case.TagsUpdated` fires for both adding and removing a tag, so diff the tag set rather than
inferring direction; and `Deleted` events do **not** cascade to sub-entities, so on `Case.Deleted`
you must fan out the deletion of its companies, individuals and checks in your own store.

## 4 — Act on `Check.ReviewNeeded`

This is the operationally significant event: a vendor could not decide alone. Route it to a queue.

Record the verdict with `PATCH /v1/checks/{type}/{id}/review`,
`{"action": "approve" | "reject", "comment": "..."}`, `override: true` to change a prior decision.
The reviewer is recorded as `api` and the same `Check.Approved` / `Check.Rejected` events fire as
for a console decision — so guard against re-processing your own write.

Per-type operations: `document-review`, `id-document-review`, `id-verification-review`,
`ekyc-review`, `electronic-signature-review`, `fraud-database-review`,
`online-reputation-review`.

**AML is a two-step.** `aml-review-hits` must confirm or ignore every hit before `aml-review` will
accept a verdict on the check; the wrong order returns `400`. `aml-update-monitoring` toggles
ongoing monitoring on an AML check.

`document-force-review` forces manual review of a Document check sitting `in_progress` with files
attached — the escape hatch when automated analysis stalls.

**Company monitoring is read-only over the API** (`company-monitoring-get-one` only); its review
happens in the console.

## 5 — Reconcile what you missed

Deliveries are logged. `webhook-log-get-many` (`GET /v1/webhook-logs`) returns every event
delivered for the workspace with its full payload and each delivery attempt with its response.

- filterable by case, individual, event and date
- default sort `created_at.desc`, `limit` capped at **50** (not the usual 100)
- retention **30 days** by default — older events are gone, so a reconciliation window longer than
  30 days must be built on `case-get-many` / `check-get-many` with `updated_at.gt`, not on the log

Run this on every startup and after any endpoint outage. A targeted retry is also available for a
failed delivery.

## 6 — Autonomy runs

`routine-trigger-routine` (`POST /v1/routines/{slug}/trigger`) enqueues an Autonomy routine and
returns the `thread_id`. The terminal outcome arrives as `AutonomyChatRun.Completed` (carrying
`thread_id`, `run_id`, `outcome`, `last_message`, `routine_id`, `routine_slug`) or
`AutonomyChatRun.Failed` (`aborted` or `error`). Correlate on `thread_id` — no lookup needed.

Only a routine carrying an `external_api` trigger can be fired this way; a routine without one, or a
disabled routine, returns `409`. That `409` is a configuration fault, not a transient error — do not
retry it.

## Rate limits while reconciling

800 `GET`/min and 300 write/min **per workspace API key**, bursts 200 and 100. A backfill over
`webhook-log-get-many` will hit the read limit before anything else. On `429` back off — there is no
`Retry-After` and no `RateLimit-*` header to read a budget from, so pace deliberately (roughly 13
reads/second sustained) rather than sprinting into the limiter.
