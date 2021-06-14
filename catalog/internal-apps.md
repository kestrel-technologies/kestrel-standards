# Internal Applications Catalog — Kestrel Vantage view

**Page owner:** CorpEng, Business Systems · **Catalog policy:** every application that stores or processes Kestrel data must have an entry, an owner team and a named contact · **Last full review:** 2026-01 (annual) · **Classification:** Kestrel Internal

## How to read this page

- **Owner team / contact** is the team accountable for the application and the person who answers questions about it. "Legacy" owners are carried over from an acquired company's records and may be out of date; contact the business unit's systems director.
- **Data sensitivity** uses the KSS scale: **Kestrel Restricted** (customer or employee personal data, contracts, pricing), **Kestrel Internal**, **Public**.
- **Vantage onboarding status** describes whether the application is in use by Kestrel Vantage and, for Kestrel platforms, whether Vantage's systems are connected to it.
- **Kestrel controls status** is **self-reported by the application owner** at the last catalog review. It is an attestation, not a verification. Where EntSec has performed an assessment, the assessment result is linked and takes precedence (KSS §2).
- Engagement note for Castellan Partners: consultants reach Roost, Charter, Aviary, Perch and the Vantage Salesforce orgs through the Castellan gateway; the actual endpoints for this engagement are listed under **Systems Access** in the Castellan Field Console. URLs below are the Kestrel-internal patterns.

## 1. Kestrel platforms

### Roost — source control and code review

| Field | Value |
|---|---|
| Purpose | Source control, changelists (CLs) and code review for every Kestrel repository that produces a production artifact |
| Owner team / contact | CorpEng, Developer Platform · Perch queue `ROOST` |
| URL pattern | `https://roost.kestrel.example/<org>/<repo>` |
| Data sensitivity | Kestrel Internal (source); repositories must not contain Restricted data |
| Vantage onboarding status | **Partial.** `vantage-salesforce` onboarded 2022-09; `birdseye` mirrored 2022-11 (read-only, no code owner). Vantage internal tools other than these are not in Roost. |
| Kestrel controls status (self-reported, CorpEng Developer Platform, 2026-01) | SSO: Enforced (Flock) · Monitoring: Onboarded (Watchtower) · Release: Flightpath, enforced · Secrets: Strongbox |

### Flightpath — release process

| Field | Value |
|---|---|
| Purpose | The gated path from `develop` to `uat` to `main` for services, infrastructure and Salesforce metadata; runs validation deploys and the Flightpath Reviewer |
| Owner team / contact | Office of the CTO, Engineering Standards · Daniel Reyes (Director) · tooling: CorpEng Release Engineering |
| URL pattern | `https://flightpath.kestrel.example/cl/<repo>/<number>` |
| Data sensitivity | Kestrel Internal |
| Vantage onboarding status | **Partial.** Gates 1 and 2 enforced for `vantage-salesforce`; **Gate 3 (production) advisory until Q1 2027** per the Vantage integration plan. See *Flightpath §10*. |
| Kestrel controls status (self-reported, Office of the CTO, 2026-03) | SSO: Enforced · Monitoring: Onboarded · Release: n/a (is the release system) · Secrets: Strongbox |

### Flock — identity lifecycle and SaaS provisioning

| Field | Value |
|---|---|
| Purpose | Kestrel's identity lifecycle platform: joiner / mover / leaver processing from HR, single sign-on federation to SaaS platforms, provisioning and deprovisioning of SaaS accounts through per-tenant connectors |
| Owner team / contact | EntSec, Identity Platforms · Samuel Oduya (Director) · Perch queue `FLOCK` |
| URL pattern | `https://flock.kestrel.example/tenants/<tenant>` |
| Data sensitivity | Kestrel Restricted (employee identity data) |
| Vantage onboarding status | **Configured.** Salesforce connector configured for Vantage Sales & Service 2023-02 (UAT and production tenants). Production tenant: see CHG-2291 (2025-05). Other Vantage SaaS (see §3): not connected. |
| Kestrel controls status (self-reported, EntSec Identity Platforms, 2026-01) | SSO: Enforced (is the identity provider) · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: Strongbox |

### Watchtower — central event monitoring

| Field | Value |
|---|---|
| Purpose | Central collection, retention (12 months) and alerting for authentication, administrative, API and data-export events from every in-scope platform (KSS-MON-07) |
| Owner team / contact | EntSec, Detection & Monitoring · Lin Zhao (Director) · Perch queue `WT` |
| URL pattern | `https://watchtower.kestrel.example/sources/<source-id>` |
| Data sensitivity | Kestrel Restricted (event data includes user identifiers) |
| Vantage onboarding status | **Pending.** Vantage Sales & Service production: Perch `WT-3317`. UAT source added 2026-10 for collector testing. Birdseye, Pulsar and the Skyward-era SaaS in §3: not onboarded. |
| Kestrel controls status (self-reported, EntSec Detection & Monitoring, 2026-01) | SSO: Enforced · Monitoring: n/a (is the monitoring system) · Release: Flightpath, enforced · Secrets: Strongbox |

### Aviary — people directory

| Field | Value |
|---|---|
| Purpose | Employee directory and org chart: name, title, team, location, manager, Chirp handle. Fed from HR nightly. |
| Owner team / contact | CorpEng, People Systems · Perch queue `AVIARY` |
| URL pattern | `https://aviary.kestrel.example/people/<handle>` · `.../teams/<team>` |
| Data sensitivity | Kestrel Internal (work contact data only) |
| Vantage onboarding status | **Complete** for employees (migrated 2021-06). Contractors and agency staff are not listed. Transfers appear after the next HR cycle. |
| Kestrel controls status (self-reported, CorpEng People Systems, 2026-01) | SSO: Enforced · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: Strongbox |

### Charter — policy and standards portal

| Field | Value |
|---|---|
| Purpose | Publishes Kestrel standards (KSS, KES), processes (Flightpath, incident management), this catalog and recent-incident summaries. Generated from the `kestrel-standards` repository. |
| Owner team / contact | EntSec Governance (Hannah Kowalczyk) for KSS; Office of the CTO (Daniel Reyes) for KES and Flightpath; CorpEng Business Systems for this catalog |
| URL pattern | `https://charter.kestrel.example/<document>` (`/kss`, `/kes`, `/flightpath`, `/catalog`, `/incidents`) |
| Data sensitivity | Kestrel Internal |
| Vantage onboarding status | **Complete** (read access for all Vantage staff; Castellan access via gateway) |
| Kestrel controls status (self-reported, EntSec Governance, 2026-01) | SSO: Enforced · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: none held |

### Perch — ticketing and issue tracking

| Field | Value |
|---|---|
| Purpose | Tickets for incidents (`INC-####`), changes (`CHG-####`), remediation items (`REM-###`), security exceptions (`ENTSEC-EXC-####`), architecture reviews (`KSS-REV-####`), engineering exceptions (`KES-EXC-####`) and team work queues (for example `WT-####` for Watchtower) |
| Owner team / contact | CorpEng, Business Systems · Perch queue `PERCH` |
| URL pattern | `https://perch.kestrel.example/<queue>/<ticket-id>` |
| Data sensitivity | Kestrel Internal; tickets must not contain credentials or customer personal data (KSS-SEC-04) |
| Vantage onboarding status | **Complete.** Vantage queues migrated from Skyward's tracker 2022-04; Skyward-era tickets imported read-only. |
| Kestrel controls status (self-reported, CorpEng Business Systems, 2026-01) | SSO: Enforced · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: Strongbox |

### Chirp — internal chat

| Field | Value |
|---|---|
| Purpose | Direct messages, channels and group conversations for all Kestrel staff and approved contractors and consultants |
| Owner team / contact | CorpEng, Collaboration · Perch queue `CHIRP` |
| URL pattern | `https://chirp.kestrel.example` |
| Data sensitivity | Kestrel Internal; do not paste Restricted data or credentials |
| Vantage onboarding status | **Complete** (2021-05). Channels in use by Vantage include `#vantage-salesforce`, `#vantage-remediation`, `#vantage-support-l2`. |
| Kestrel controls status (self-reported, CorpEng Collaboration, 2026-01) | SSO: Enforced · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: Strongbox |

### Strongbox — secrets vault

| Field | Value |
|---|---|
| Purpose | Approved storage for credentials, API keys, tokens and certificates, with rotation reminders and access audit. For Salesforce, the approved equivalent is Named Credentials backed by External Credentials. |
| Owner team / contact | EntSec, Identity Platforms · Samuel Oduya (Director) · Perch queue `STRONGBOX` |
| URL pattern | `https://strongbox.kestrel.example/vaults/<team>/<secret>` |
| Data sensitivity | Kestrel Restricted |
| Vantage onboarding status | **Available; Vantage integrations not migrated.** Vantage has a team vault (created 2022-03). As of the last review, the Birdseye integration credential is managed by Sales Ops outside Strongbox; the Pulsar callout uses a Salesforce Named Credential. |
| Kestrel controls status (self-reported, EntSec Identity Platforms, 2026-01) | SSO: Enforced · Monitoring: Onboarded · Release: Flightpath, enforced · Secrets: n/a |

## 4. Requesting a change to this catalog

Open a Perch ticket in the `CATALOG` queue with the application name, the field to change, and evidence. Owner and contact changes require the business unit's systems director to confirm. Control-status fields are re-attested by the owner annually; an EntSec assessment replaces the self-reported value.
