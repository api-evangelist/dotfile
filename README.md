# Dotfile

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Dotfile is a French (Paris-based) end-to-end business verification platform that automates KYB, KYC and AML compliance for regulated and high-growth companies. A single REST API and a case-management console act on the same data: a case holds the companies and individuals under verification, the checks run on them, and the decision that closes it. Nine check types are exposed — document, id_document, id_verification, ekyc, aml, company_monitoring, electronic_signature, fraud_database and online_reputation — alongside company-data search and fetch against official registries, document ordering, risk scoring, periodic review, templates, tables, custom properties, white-label client portals, tags, activities and an autonomous "Autonomy" agent routine trigger. The API is 100 operations over 82 paths, keyed with an X-DOTFILE-API-KEY header, and reports check outcomes asynchronously through 46 subscribable webhook events.

**Areas:** KYB · KYC · AML · Business Verification · Identity Verification · Compliance · RegTech · Onboarding · Sanctions Screening · Document Verification · Fraud Prevention · Company Data

## Dotfile on APIs.io

Everything profiled for Dotfile is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/dotfile/](https://apis.io/providers/dotfile/)** — or go straight to one of the APIs or properties below.

### APIs (26)

- [Dotfile Activities API](https://apis.io/apis/dotfile/dotfile-activities-api/) — The Activities API from Dotfile — 1 operation(s) for activities.
- [Dotfile AML check API](https://apis.io/apis/dotfile/dotfile-aml-check-api/) — The AML check API from Dotfile — 5 operation(s) for aml check.
- [Dotfile Cases API](https://apis.io/apis/dotfile/dotfile-cases-api/) — The Cases API from Dotfile — 10 operation(s) for cases.
- [Dotfile Checks API](https://apis.io/apis/dotfile/dotfile-checks-api/) — The Checks API from Dotfile — 2 operation(s) for checks.
- [Dotfile Client portal API](https://apis.io/apis/dotfile/dotfile-client-portal-api/) — The Client portal API from Dotfile — 3 operation(s) for client portal.
- [Dotfile Companies API](https://apis.io/apis/dotfile/dotfile-companies-api/) — The Companies API from Dotfile — 4 operation(s) for companies.
- [Dotfile Company data API](https://apis.io/apis/dotfile/dotfile-company-data-api/) — The Company data API from Dotfile — 6 operation(s) for company data.
- [Dotfile Company Monitoring check API](https://apis.io/apis/dotfile/dotfile-company-monitoring-check-api/) — The Company Monitoring check API from Dotfile — 1 operation(s) for company monitoring check.
- [Dotfile Custom properties API](https://apis.io/apis/dotfile/dotfile-custom-properties-api/) — The Custom properties API from Dotfile — 2 operation(s) for custom properties.
- [Dotfile Document check API](https://apis.io/apis/dotfile/dotfile-document-check-api/) — The Document check API from Dotfile — 6 operation(s) for document check.
- [Dotfile eKYC check API](https://apis.io/apis/dotfile/dotfile-ekyc-check-api/) — The eKYC check API from Dotfile — 3 operation(s) for ekyc check.
- [Dotfile Electronic Signature check API](https://apis.io/apis/dotfile/dotfile-electronic-signature-check-api/) — The Electronic Signature check API from Dotfile — 3 operation(s) for electronic signature check.
- [Dotfile Files API](https://apis.io/apis/dotfile/dotfile-files-api/) — The Files API from Dotfile — 2 operation(s) for files.
- [Dotfile Fraud database check API](https://apis.io/apis/dotfile/dotfile-fraud-database-check-api/) — The Fraud database check API from Dotfile — 3 operation(s) for fraud database check.
- [Dotfile ID Document check API](https://apis.io/apis/dotfile/dotfile-id-document-check-api/) — The ID Document check API from Dotfile — 4 operation(s) for id document check.
- [Dotfile ID Verification check API](https://apis.io/apis/dotfile/dotfile-id-verification-check-api/) — The ID Verification check API from Dotfile — 4 operation(s) for id verification check.
- [Dotfile Individuals API](https://apis.io/apis/dotfile/dotfile-individuals-api/) — The Individuals API from Dotfile — 3 operation(s) for individuals.
- [Dotfile Notes API](https://apis.io/apis/dotfile/dotfile-notes-api/) — The Notes API from Dotfile — 2 operation(s) for notes.
- [Dotfile Online Reputation check API](https://apis.io/apis/dotfile/dotfile-online-reputation-check-api/) — The Online Reputation check API from Dotfile — 3 operation(s) for online reputation check.
- [Dotfile Ping API](https://apis.io/apis/dotfile/dotfile-ping-api/) — The Ping API from Dotfile — 1 operation(s) for ping.
- [Dotfile Routines API](https://apis.io/apis/dotfile/dotfile-routines-api/) — The Routines API from Dotfile — 1 operation(s) for routines.
- [Dotfile Tables API](https://apis.io/apis/dotfile/dotfile-tables-api/) — The Tables API from Dotfile — 2 operation(s) for tables.
- [Dotfile Tags API](https://apis.io/apis/dotfile/dotfile-tags-api/) — The Tags API from Dotfile — 3 operation(s) for tags.
- [Dotfile Templates API](https://apis.io/apis/dotfile/dotfile-templates-api/) — The Templates API from Dotfile — 3 operation(s) for templates.
- [Dotfile Users API](https://apis.io/apis/dotfile/dotfile-users-api/) — The Users API from Dotfile — 2 operation(s) for users.
- [Dotfile Webhooks API](https://apis.io/apis/dotfile/dotfile-webhooks-api/) — The Webhooks API from Dotfile — 3 operation(s) for webhooks.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [Dotfile Webhooks](https://apis.io/asyncapis/dotfile/dotfile-webhooks/)

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [dotfile](https://apis.io/servers/dotfile/dotfile/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Dotfile Agentic Access](https://apis.io/agentic-access/dotfile/dotfile-agentic-access/)

#### Overlays (1)

OpenAPI overlays, including conversational phrasing for each operation.

- [Dotfile Api Overlay](https://apis.io/overlays/dotfile/dotfile-api-overlay/)

#### Security (3)

Authentication, trust centre, and vulnerability disclosure.

- [Dotfile Authentication](https://apis.io/security/dotfile/dotfile-authentication/)
- [Dotfile Domain Security](https://apis.io/security/dotfile/dotfile-domain-security/)
- [Dotfile Trust Center](https://apis.io/security/dotfile/dotfile-trust-center/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Dotfile Plans Pricing](https://apis.io/plans/dotfile/dotfile-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Dotfile Rate Limits](https://apis.io/rate-limits/dotfile/dotfile-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.dotfile.com/)
- [Documentation](https://docs.dotfile.com/)
- [Getting started](https://docs.dotfile.com/docs/quick-start)
- [API reference](https://docs.dotfile.com/reference/overview)
- [Support](https://docs.dotfile.com/reference/getting-help)
- [Blog](https://www.dotfile.com/resources)
- [Compliance](https://trust.dotfile.com/)
- [Terms of service](https://www.dotfile.com/terms)
- [Privacy policy](https://www.dotfile.com/privacy)
- [GitHub](https://github.com/DotfileTech)
- [Status](https://status.dotfile.com/)

## API Evangelist rating

**Kin Score 51.2** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 78.6 |
| Contract quality | 60.3 |
| Contract governance | 4.5 |
| Operational transparency | 59.2 |
| Developer ergonomics | 47.0 |
| Access clarity | 35.5 |

**Agent readiness 33.2** — agent-ready

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | — |
| MCP Server | — |
| Auth Clarity | bearer (partial credit) |
| Idempotency | — |
| Error Semantics | derived (partial credit) |
| OpenAPI Examples | partial |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | yes |
| Agent Skills | derived (partial credit) |
| Well Known Catalog | yes |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | partial |
| Operation Distinctiveness | distinct |

**Regulatory alignment 24.5** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/dotfile/](https://apis.io/providers/dotfile/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Dotfile: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/dotfile/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
