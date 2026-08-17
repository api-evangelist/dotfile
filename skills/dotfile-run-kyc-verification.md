---
name: Run a KYC verification on Dotfile
description: Open a Dotfile case holding one individual, let the template create the identity and screening checks, collect the document and selfie through the client portal, then record verdicts and close the case.
api: openapi/_original/dotfile-openapi.json
generated: '2026-08-17'
method: generated
source: https://docs.dotfile.com/reference/start-kyc-process
operations:
  - case-create-one
  - client-portal-share-client-portal-link
  - case-get-one
  - id-verification-get-one
  - id-verification-refresh-url
  - id-verification-review
  - aml-review-hits
  - aml-review
  - case-review-create-one
---

# Run a KYC verification on Dotfile

A KYC process is the **same object** as a KYB one — a case — with one individual in it instead of a
company and its ownership chain. There is no registry step, which makes this the shorter flow.

Base URL `https://api.dotfile.com/v1`, key in `X-DOTFILE-API-KEY`, HTTPS only. No sandbox exists;
every check dispatches to a paid vendor.

## 1 — Create the case and the individual in one call

`case-create-one` (`POST /v1/cases`). Omit `companies` entirely — a case does not need one.

```json
{
  "name": "Jane Doe onboarding",
  "external_id": "customer_8412",
  "template_key": "kyc_standard",
  "individuals": [
    { "first_name": "Jane", "last_name": "Doe",
      "email": "jane.doe@example.com",
      "birth_date": "1980-04-12",
      "birth_country": "FR",
      "nationalities": ["FR"],
      "is_business_contact": true }
  ]
}
```

- `email` is **required** when `is_business_contact` is `true`. That flag is what makes the person
  reachable, both for the client portal and for the checks that send them somewhere.
- `external_id` is your identifier, unique per workspace, and works in place of the case id on read.
- The response is the complete case, including the id Dotfile assigned to the individual.
- Templates, custom properties, tags and risk behave exactly as on a KYB case.

## 2 — Know which checks can target a person

The template decides which run, but not every check type accepts an individual:

| Check | Individual | Company |
|---|---|---|
| Identity verification, ID document, eKYC, electronic signature | yes | no |
| AML screening, document, fraud database | yes | yes |
| Online reputation, company monitoring | no | yes |

A KYC template built on identity verification + AML screening covers the usual case. If a template
would create a company-only check on an individual, that is a template problem to fix in the
console, not something to work around by hand.

## 3 — Collect what only the person can give

An identity verification needs their document and their face. Generate the portal link with
`client-portal-share-client-portal-link` (`POST /v1/cases/{id}/share-client-portal-link`,
`{"business_contact_id": "<individual id>"}`). The individual must belong to the case, be relevant,
and have an email; the portal must be online.

If you drive your own front end instead, upload with `file-upload-file`
(`POST /v1/files/upload`, max 20MB, `413` above that, `415` on an unsupported type) and pass the
returned `upload_ref` to `id-document-create-one` or `document-create-one`. An `upload_ref` is
valid for about **one day**.

For a hosted identity verification, `id-verification-get-one` returns
`data.vendor.verificationUrl` — the vendor's own page. It expires; mint a fresh one valid 15 minutes
with `id-verification-refresh-url` rather than caching the URL.

## 4 — Receive the results

Subscribe with `webhook-create-one` to `Check.Started`, `Check.Approved`, `Check.Rejected`,
`Check.ReviewNeeded`, `Check.Expired`, plus `Case.RiskUpdated` and `Case.StatusUpdated`. Dotfile
publishes no webhook signature scheme — do not trust a delivery body on its own; re-read the case
with `case-get-one` before acting on anything consequential.

`case-get-one` returns the individual with their `checks` embedded, addressable by your
`external_id`.

`Check.ReviewNeeded` means the vendor could not decide alone — a blurred document, a partial
sanctions match.

## 5 — Review, then decide

Per check type, `PATCH /v1/checks/{type}/{id}/review` with
`{"action": "approve" | "reject", "comment": "..."}`, `override: true` to change a prior verdict.
The reviewer is recorded as `api`, and the same `Check.Approved` / `Check.Rejected` events fire as
for a console decision.

- `id-verification-review`, `id-document-review`, `ekyc-review`, `document-review`,
  `electronic-signature-review`, `fraud-database-review`
- AML: `aml-review-hits` for every hit FIRST, then `aml-review`. Out of order returns `400`.

Close with `case-review-create-one` (`POST /v1/cases/{id}/reviews`),
`{"status": "approved", "comment": "..."}`. Auto-approval on the template does this once every
check it created is approved. `Case.ReviewConfirmed` ends the process.

## Judgement rule for an agent

Recording a verdict over the API records the DECISION but not the EVIDENCE the console would show a
human — the document beside the extracted fields, the partial match beside the identity it matched.
On a `Check.ReviewNeeded` that turns on ambiguous evidence, escalate to a person rather than
approving. `Check.ReviewNeeded` exists precisely because a machine already declined to decide.

## Errors

Same catalog as the KYB skill; see `errors/dotfile-problem-types.yml`. Retry only `429`, `500`,
`502`. No idempotency key exists, so read back by `external_id` before retrying any write.
