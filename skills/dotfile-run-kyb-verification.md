---
name: Run a KYB verification on Dotfile
description: Identify a company in an official registry, open a Dotfile case carrying the company, its ownership chain and the people behind it, let the template create the checks, then follow the results to a decision.
api: openapi/_original/dotfile-openapi.json
generated: '2026-08-17'
method: generated
source: https://docs.dotfile.com/reference/start-kyb-process
operations:
  - company-data-countries
  - company-data-search
  - company-data-fetch
  - case-create-one
  - client-portal-share-client-portal-link
  - case-get-one
  - case-review-create-one
---

# Run a KYB verification on Dotfile

Base URL `https://api.dotfile.com/v1`. Every request carries `X-DOTFILE-API-KEY: <workspace key>`.
HTTPS only — plain HTTP fails. There is no sandbox and no test-mode key: a Dotfile key is a
production credential against a real workspace, and every check you create dispatches to a paid
third-party vendor. Do not create checks speculatively.

## Before you start

You need a workspace API key and a **template key** (for example `kyb_standard`). List templates
with `template-get-many` and read one with `template-get-one` if you do not know which key to use.
The template — not you — decides which checks are created, and on which entities.

Smoke-test the key with `ping-ping` (`GET /v1/ping`). It is the only zero-side-effect call in the
API and returns `workspace_name`.

## 1 — Identify the company

Skip to step 2 if you already hold a trusted country + registration number: pass them straight to
the case and Dotfile fetches the rest.

Otherwise:

1. `company-data-countries` (`GET /v1/company-data/countries`) — the countries your workspace's
   configured vendors actually cover. Check this before promising coverage.
2. `company-data-search` (`GET /v1/company-data/search`) — query by `country` and `name`.
3. `company-data-fetch` (`GET /v1/company-data/fetch/{search_ref}`) — turn a result's `search_ref`
   into the full profile: legal form, address, classifications, shareholders, officers.

**Never take the first search result.** Two companies can share a name, in the same country, in the
same registry. Either a human selects from the candidate list, or you match a registration number
you already trust. Auto-selecting means onboarding a company your customer never named — an
unrecoverable compliance error, not a UX detail. If you cannot disambiguate, stop and ask.

`company-data-fetch` can return `400` with "Limit reached. Contact us at support@dotfile.com to lift
all limits." That is a quota failure dressed as a validation error. Do not retry it.

## 2 — Create the case, its entities and its checks in ONE call

`case-create-one` (`POST /v1/cases`) accepts the whole graph in one transaction: the main company,
affiliated companies, individuals, and the relations between them.

```json
{
  "name": "Acme Corp onboarding",
  "external_id": "customer_8412",
  "template_key": "kyb_standard",
  "companies": [
    { "ref": "main-co", "type": "main", "name": "Acme Corp",
      "country": "FR", "registration_number": "552100554" },
    { "ref": "holding", "type": "affiliated", "name": "Acme Holding",
      "country": "LU", "registration_number": "B123456",
      "relations": [
        { "source_company_ref": "main-co", "roles": ["shareholder"], "ownership_percentage": 80 }
      ] }
  ],
  "individuals": [
    { "ref": "ceo", "first_name": "Jane", "last_name": "Doe",
      "email": "jane.doe@acme.com", "birth_date": "1980-04-12",
      "is_business_contact": true, "is_beneficial_owner": true,
      "relations": [
        { "source_company_ref": "main-co",
          "roles": ["legal_representative", "shareholder"],
          "ownership_percentage": 20, "position": "CEO" }
      ] }
  ]
}
```

Rules that will bite you otherwise:

- **`ref` is yours to invent.** No ids exist yet, so entities reference each other inside the
  payload by `ref` and Dotfile resolves them.
- **`template_key` creates the checks.** Do not call `POST /v1/checks/{type}` yourself as part of
  this flow.
- **`roles` is a property of the RELATION** and takes only `legal_representative` or `shareholder`.
  Beneficial ownership is a property of the PERSON: `is_beneficial_owner`, alongside
  `is_controlling_person`, `is_signatory`, `is_delegator`, `is_business_contact`.
- **`source_company_ref`** names the company at the far end of a relation. Omit it and the case's
  main company is used, which is usually what you want.
- **`external_id`** is your identifier, unique per workspace, and is accepted in place of the
  Dotfile id when reading the case back — so you never have to store a Dotfile UUID.
- **A case has exactly one `main` company.** Sending two, or none, returns `400`.

The response is the complete case, with every id Dotfile assigned. No follow-up read is needed.

There is **no idempotency key** in this API. `POST /v1/cases` retried after a timeout can create a
second case. Before retrying, read back with `case-get-one` using your `external_id`.

The older sequence — create the case, then `company-create-one` / `individual-create-one` against
its `case_id` — still works and is not deprecated, but it is more round-trips and permits partial
failures. Prefer the nested form.

## 3 — Collect what the registry does not hold

A registry gives you structure, not a proof of address or a selfie. That comes from the customer
through the Client Portal.

`client-portal-share-client-portal-link` (`POST /v1/cases/{id}/share-client-portal-link`) with
`{"business_contact_id": "<individual id>"}` returns an authenticated link.

Preconditions, all of which return `400` when unmet: the individual belongs to the case, is
relevant, and has an email address; the client portal is online. Setting
`is_business_contact: true` at creation is what makes a person eligible.

If the portal has a `wait` step, `Case.ClientPortalWaitStepTriggered` fires once and the case
parks. Resume it with `client-portal-complete-client-portal-wait-step`.

## 4 — Receive the results

Checks are asynchronous. Prefer webhooks; poll only as a fallback.

- **Webhooks** — subscribe with `webhook-create-one` to `Check.Started`, `Check.Approved`,
  `Check.Rejected`, `Check.ReviewNeeded`, `Check.Expired`, plus `Case.RiskUpdated` and
  `Case.StatusUpdated`. Note that Dotfile publishes **no webhook signature scheme**, so
  authenticate deliveries by another means (a secret path segment, an allowlist, or by treating the
  webhook as a hint and re-reading the case).
- **Read the case** — `case-get-one` returns companies and individuals with their `checks` embedded,
  by case id or by your `external_id`. `data_lineage=true` adds where each field value came from
  (this parameter replaced `property_origin` on 2026-08-14; the old name now returns `200` and
  silently omits the lineage).

`Check.ReviewNeeded` is the event that matters: a vendor could not decide alone.

## 5 — Record verdicts, then decide

Per-check review is `PATCH /v1/checks/{type}/{id}/review` with
`{"action": "approve" | "reject", "comment": "..."}`; `override: true` changes a verdict already
made. The reviewer is recorded as `api`.

- `document-review`, `id-document-review`, `id-verification-review`, `ekyc-review`,
  `electronic-signature-review`, `fraud-database-review`, `online-reputation-review`
- AML: review every hit with `aml-review-hits` FIRST, then `aml-review`. Reviewing the check with
  hits outstanding returns `400`.
- Company monitoring is read-only over the API (`company-monitoring-get-one` only).

Close the case with `case-review-create-one` (`POST /v1/cases/{id}/reviews`),
`{"status": "approved", "comment": "..."}`. `comment` may be mandatory depending on workspace
settings, and `next_review_at` is computed from the workspace settings and the case risk level
unless you override it — that is how periodic review gets scheduled. A template with auto-approval
closes the case for you once every check it created is approved. Either way,
`Case.ReviewConfirmed` is the signal the process is over.

## Error and retry rules

Read `errors/dotfile-problem-types.yml` for the full catalog. The envelope is
`{"status_code","timestamp","code","message"}` — NOT RFC 9457, and not declared in the OpenAPI.

| Status | Retry? | Notes |
|---|---|---|
| 400 | No | Malformed body, non-UUID path id, bad filter/sort/page, or a business-rule violation. Also how quota exhaustion ("Limit reached") is reported. |
| 401 | No | Missing or invalid `X-DOTFILE-API-KEY`. |
| 403 | No | Your IP is outside the allowlist configured on that key. |
| 404 | No | Wrong id, or the id belongs to another workspace. |
| 409 | No | Resource state forbids the action. |
| 429 | Yes | Back off. 800 GET/min and 300 write/min per key, burst 200/100. **No `Retry-After` or `RateLimit-*` header is published**, so choose your own interval — start at a few seconds and widen. |
| 500 | Yes | Reported automatically by Dotfile; no ticket needed. |
| 502 | Yes | A third-party vendor in the request path is unavailable. |

Never retry a write without first reading back by `external_id`: there is no idempotency key.
