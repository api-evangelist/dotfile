---
name: Search registries and order company documents on Dotfile
description: Resolve a company against official registries, pull its full profile and vendor data, then order and retrieve official documents such as a French KBIS or annual accounts.
api: openapi/_original/dotfile-openapi.json
generated: '2026-08-17'
method: generated
source: https://docs.dotfile.com/reference/company-data-guide
operations:
  - company-data-countries
  - company-data-entity-legal-forms
  - company-data-search
  - company-data-fetch
  - company-get-one-vendor-data
  - company-data-get-available-documents
  - company-data-document-order-create-one
  - case-get-document
  - file-get-file
---

# Search registries and order company documents on Dotfile

This is the registry side of Dotfile, usable independently of a verification case: resolve a legal
entity, read what the registry holds, and buy the official documents.

Base URL `https://api.dotfile.com/v1`, key in `X-DOTFILE-API-KEY`. Every call here consumes vendor
credit — Creditsafe, Kyckr, INPI and others depending on workspace configuration — so cache
aggressively and do not loop.

## 1 — Establish coverage first

- `company-data-countries` (`GET /v1/company-data/countries`) — the countries available **with the
  vendors configured in your workspace**. Two workspaces will not return the same list. Check this
  before telling a user a country is supported.
- `company-data-entity-legal-forms` (`GET /v1/company-data/entity-legal-forms`) — the supported
  entity legal forms, per **ISO 20275** (the GLEIF ELF code list). Use these codes rather than
  free-text legal-form strings.

## 2 — Search, then fetch

`company-data-search` (`GET /v1/company-data/search`) takes a `country` plus a `name` or a
`registration_number`. Each result carries a `search_ref`.

`company-data-fetch` (`GET /v1/company-data/fetch/{search_ref}`) turns one `search_ref` into the
full profile: legal form, address, classifications, shareholders and officers.

**Disambiguation is the caller's job.** Search deliberately returns several candidates because two
companies can share a name inside one registry. Pick by a registration number you already trust, or
put the candidate list in front of a human. Never auto-select the first hit.

`company-data-fetch` returns `400` when the workspace fetch quota is exhausted, with the message
"Limit reached. Contact us at support@dotfile.com to lift all limits." That is not transient — do
not retry it, and surface it as a billing/limit condition rather than a bad request.

For a company already on a case, `company-get-one-vendor-data`
(`GET /v1/companies/{id}/vendor-data`) returns the raw vendor payload behind the Dotfile view —
what to reach for when you need to explain or audit where a value came from.

## 3 — See what documents exist

`company-data-get-available-documents` (`GET /v1/company-data/available-documents`) requires
**either** `company_id` **or** `country` + `registration_number`. Sending neither, or an incomplete
pair, returns `400`.

Each entry names the `vendor` that can supply it (values include `inpi`, added 2026-07-21). What is
available is country- and vendor-specific: a French KBIS, annual accounts, articles of association.

## 4 — Order, then wait for the webhook

`company-data-document-order-create-one` (`POST /v1/company-data/document-orders`) places the order.
It is **asynchronous** and costs money.

Outcome events:

- `DocumentOrder.Completed` — the document was retrieved (for example a KBIS for a French company)
- `DocumentOrder.Failed` — the document is unavailable (for example annual accounts never filed)

`DocumentOrder.Failed` is a normal, expected outcome for a registry that simply does not hold the
document. Do not retry it as if it were an error; some documents do not exist for some companies.

Subscribe to both with `webhook-create-one` before ordering. Do not poll.

## 5 — Retrieve the file

- `case-get-document` (`GET /v1/cases/{id}/documents`) — every document attached to a case,
  including completed orders.
- `file-get-file` (`GET /v1/files/{id}`) — the bytes, as `binary/octet-stream`. `404` if the file id
  is unknown to the workspace.

## Interaction with case creation

If you already hold a trusted `country` + `registration_number`, you do not need steps 2–3 at all:
pass them straight into `case-create-one` and Dotfile fetches the registry data itself. Use this
skill when you need the company profile **before** deciding to open a case, when you need the raw
vendor payload for an audit trail, or when you need an official document as evidence.

## Errors

| Status | Meaning here | Retry? |
|---|---|---|
| 400 | Missing `company_id` or `country`+`registration_number`; unsupported country; **or fetch quota exhausted** | No |
| 401 | Missing/invalid `X-DOTFILE-API-KEY` | No |
| 403 | Caller IP outside the key's allowlist | No |
| 404 | Unknown `search_ref`, company id or file id | No |
| 429 | 800 reads/min per key exceeded | Yes, with self-chosen backoff — no `Retry-After` is published |
| 502 | A registry vendor is unavailable | Yes |

Cache `company-data-countries` and `company-data-entity-legal-forms`; they change rarely and each
call is wasted budget.
