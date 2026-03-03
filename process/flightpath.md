# Flightpath — Release Process for Production Changes (v2.3)

**Document:** FP-2.3 · **Owner:** Office of the CTO, Engineering Standards (Daniel Reyes) · **Tooling owner:** CorpEng, Release Engineering · **Effective:** March 3, 2026 · **Classification:** Kestrel Internal

## 0. Scope

1. Flightpath is the only path by which a change reaches a Kestrel production system. It applies to Kestrel-built services, infrastructure definitions and Salesforce metadata alike.
2. It applies to acquired entities from acquisition close. Where a business unit is still adopting Flightpath, its adoption plan (§10) states which gates are **enforced** and which are **advisory**. Advisory gates still run and still report; they do not prevent a merge.
3. Flightpath does not define *what* a change must contain. The Kestrel Engineering Standards (KES) define that; Flightpath checks it.

## 1. Vocabulary

| Term | Meaning |
|---|---|
| CL | Changelist: a pull request in Roost. One CL implements one ticket or one promotion. |
| Feature CL | A CL from a feature branch into `develop`. |
| Promotion CL | A CL from `develop` into `uat`, or from `uat` into `main`. Promotion CLs contain no new work, only what already merged to the source branch. |
| Gate | The set of checks and approvals a CL must pass to merge into a protected branch. There are three: Dev, UAT, Prod. |
| Validation deploy | For Salesforce: `sf project deploy validate` against the target org with local tests, which compiles, runs tests and reports coverage without saving anything. For services: build and test. |
| Flightpath Reviewer | The automated standards reviewer (KES §5). Posts a verdict table on every CL. |
| Code owner | The team named in `CODEOWNERS` for the changed path. |
| CHG ticket | A Perch change ticket (`CHG-####`) that records UAT sign-off, the production change window, and post-deploy verification. |

## 2. Environments and branches

| Branch | Environment | Salesforce org (Vantage) | Who merges | Deployed by |
|---|---|---|---|---|
| `develop` | Development: integration of feature work | `vantage-dev` (`kestrelvantage-dev`) | Code owner | Flightpath, automatically on merge |
| `uat` | User acceptance: product owner and QA testing against realistic data | `vantage-uat` (`kestrelvantage-uat`) | Code owner, after product-owner sign-off | Flightpath, automatically on merge |
| `main` | Production | `vantage-prod` (`kestrelvantage-prod`) | Code owner, after CHG approval | Flightpath, automatically on merge, within the change window |

Feature branches are named `<owner>/<ticket>-<slug>` (for example `priya/REM-104-birdseye-read-profile`). Hotfix branches are named `hotfix/<ticket>-<slug>` and follow §9.

```
feature branch ──CL──▶ develop ──promotion CL──▶ uat ──promotion CL──▶ main
                         │                        │                      │
                       dev org                  UAT org               prod org
                      (Gate 1)                 (Gate 2)              (Gate 3)
```

Each merge deploys to exactly one environment. Nothing is deployed to production that is not on `main`; nothing is on `main` that was not first on `uat`.

## 3. What a CL must contain

Every CL MUST have:

1. A title in the form `<type>(<area>): <summary> [<ticket>]`, for example `fix(profiles): remove Modify All Data from Marketing Integration [REM-104]`.
2. A description using the repository template, with the fields **Ticket**, **Author**, **Reviewed-by**, **Risk** (Low / Medium / High), **Rollback plan**, and **Test evidence** (what was run, where).
3. A link to the Perch ticket. CLs without a ticket are closed by the Reviewer.
4. Tests meeting KES-DEV-2 (and KES-DEV-4 for Apex).
5. For Salesforce metadata: the retrieved source of every component changed, including profiles and permission sets touched indirectly.

## 4. Gates

### Gate 1 · Merge to `develop` (Dev)

| Check | Pass condition |
|---|---|
| CI build and tests | Services: build, unit tests, lint, dependency scan green. Salesforce: validation deploy to `vantage-dev` with `RunLocalTests` succeeds; per-class coverage ≥ 85%. |
| Flightpath Reviewer | Verdict posted on the final revision; no **Block** (or a written code-owner override per KES §5). |
| Review | At least one code-owner approval; author is not the approver. |
| Ticket | Linked Perch ticket in an open state. |

On merge: automatic deploy to the dev environment. A failed deploy reverts the merge automatically and reopens the CL.

### Gate 2 · Merge to `uat` (UAT)

| Check | Pass condition |
|---|---|
| Promotion CL | Source is `develop`; contains only merged feature CLs. |
| CI | Validation deploy to `vantage-uat` succeeds (Salesforce); build and tests green (services). |
| Flightpath Reviewer | Re-run on the promotion diff; no Block. |
| Product-owner sign-off to test | Comment on the CL from the product owner: `uat-scope: confirmed` listing the tickets included. |
| Review | One code-owner approval. |

On merge: automatic deploy to UAT. QA executes the test plan on the UAT environment. **UAT sign-off** is recorded as a comment on the CHG ticket (`uat-signoff: approved by <name> on <date>`), not on the CL.

### Gate 3 · Merge to `main` (Prod)

| Check | Pass condition |
|---|---|
| Promotion CL | Source is `uat`; contains only what is on `uat`. |
| CI | Validation deploy to `vantage-prod` (check-only, `RunLocalTests`) succeeds; org-wide and per-class coverage ≥ 85%. |
| Flightpath Reviewer | No Block; every **Warn** resolved or acknowledged by the code owner in a comment. |
| CHG ticket | A `CHG-####` ticket in state **Approved**, with UAT sign-off recorded and a change window inside the next 5 business days. |
| Review | Two approvals, at least one from the code owner. |
| Window | Merge happens inside the CHG window. Outside the window the merge button is disabled. |

On merge: automatic deploy to production. The owning team MUST record **post-deploy verification** on the CHG ticket within one business day (`verified: <what was checked> by <name>`). A CHG ticket without verification after one business day is escalated to the code owner's director.

### Deploy freezes

CorpEng Release Engineering MAY pause automatic deploys for a repository under a CHG ticket (repository variable `FLIGHTPATH_DEPLOY_ENABLED` set to `false`). During a freeze the gates still apply and CLs still merge; the deploy workflow records "skipped" with the CHG reference, and deploys resume in order when the freeze lifts. A merge during a freeze is therefore **not** Done (§7). Current freeze: `vantage-salesforce`, CHG-2410, for the duration of the KSS assessment of the production org.

## 5. Code owners for Vantage repositories

| Repository | Path | Code owner team | Contact |
|---|---|---|---|
| `vantage-salesforce` | `force-app/**` | Vantage Engineering (Salesforce) | Felix Okonkwo (VP Engineering, Vantage) |
| `vantage-salesforce` | `force-app/main/default/profiles/**`, `permissionsets/**`, `connectedApps/**`, `sharingRules/**`, `settings/**`, `sites/**`, `experiences/**` | CorpEng, CRM Platform | Jon Licht (Lead, CRM Platform) |
| `vantage-salesforce` | `force-app/main/default/flows/**` | Vantage Business Systems | Mikael Lindqvist (Sr. Product Manager) |
| `flock` | all | EntSec, Identity Platforms | Samuel Oduya (Director) |
| `watchtower` | all | EntSec, Detection & Monitoring | Lin Zhao (Director) |
| `birdseye` | all | **No code owner on file** (legacy; see Internal Applications Catalog) | — |
| `kestrel-standards` | `standards/KSS*` | EntSec Governance | Hannah Kowalczyk |
| `kestrel-standards` | `standards/KES*`, `process/**` | Office of the CTO, Engineering Standards | Daniel Reyes |

## 6. The Flightpath Reviewer

1. The Reviewer runs as a Roost check named `flightpath/reviewer` on every CL revision. It posts one comment titled **Flightpath Reviewer** containing the verdict table defined in KES §5, and sets the check to success (no Block), neutral (Warn only) or failure (Block).
2. The Reviewer evaluates the CL against the KES revision current on the CL's creation date.
3. Respond to a finding by changing the code, or by a code-owner override comment `flightpath: override <clause> — <reason>`. Overrides are reported weekly.
4. The Reviewer does not read tickets, chat or documentation. Evidence that lives outside the repository does not change its verdict.
5. Where a gate is advisory (§10), the check still runs and still posts; a failure does not disable the merge button.

## 7. Status vocabulary for tracking a change

When reporting the status of a ticket or remediation item, use exactly one of these four values. Each value is defined by **evidence in Roost, Flightpath and the target environment**, not by the opinion of the owning team.

| Status | Evidence required | Not sufficient |
|---|---|---|
| **Not started** | No CL for the item meets the In Dev test below. This includes: no CL exists; the CL is a draft; the CL's CI is failing (for example, blocked on a license, a dependency or a failing build); or the CL's linked Perch ticket is in state **Blocked**. | A ticket being assigned, "on the roadmap," or "config is written" in a branch nobody can deploy. |
| **In Dev** | A CL implementing the item is merged to `develop`, **or** is open against `develop` and meets all three of: marked ready for review (not a draft), CI passing on its latest revision, and its linked Perch ticket not in state Blocked. The change is deployed to the dev environment or deployable to it. Nothing is on `uat`. | "Merged" without saying where; a draft CL; an open CL with red checks; a CL whose ticket is Blocked. |
| **In UAT** | The change is merged to `uat` and deployed to the UAT environment (deploy job green). Not yet on `main`. | A change that is partially in production: report at the last stage that is fully complete, with a note. |
| **Done** | **All three:** merged to `main` **and** deployed to the production environment (Flightpath deploy job green) **and** post-deploy verification recorded on the CHG ticket. | Merged to `main` but the deploy failed; deployed but not verified; verified in UAT only. |

In short: an open CL counts as In Dev only if it is ready for review with passing CI and its linked ticket is not Blocked.

Rules:

1. Status is the **furthest stage fully reached**. A remediation item whose scope covers eight profiles and is applied to three in production is not Done; it is reported at the last stage where all eight were complete, with the partial production state noted.
2. "Merged" is not a status. State the branch.
3. A change made directly in a production environment without a CL (see §9) may be *in effect* but is not **Done** under Flightpath until it has been back-ported as a CL and verified. For reporting during an incident or an audit, state both facts: "in effect in prod (direct change, CHG-####); CL pending."
4. When the item is tracked in a repository that is not yet under Flightpath, the same four values apply using the repository's branches as evidence.

## 8. Rollback

1. Rollback is a forward operation: a revert CL against `main` (or the affected branch), reviewed under an expedited Gate 3 (one code-owner approval, Reviewer run, CHG ticket updated), deployed by Flightpath.
2. Manual edits in a production environment are not a rollback method, even to undo a bad deploy; they are direct production changes under §9.
3. Salesforce: destructive changes (deleting fields, objects or Flows) are never reverted automatically. The revert CL re-creates metadata; data lost to a destructive change is restored through a separate data-fix ticket owned by the business owner.
4. The owning team MUST decide rollback-or-fix within 30 minutes of a SEV-1 or SEV-2 production incident attributed to a deploy, and MUST record the decision on the incident ticket.
5. Rollback restores the previous `main`; it does not change `uat` or `develop`. Re-promotion after a fix follows the normal gates.

## 9. Emergency and direct production changes

1. A direct production change is any change to a production system made outside a Flightpath deploy: a Setup edit in a Salesforce production org, a console edit of infrastructure, a manual deploy, a hotfix pushed around branch protection.
2. Direct production changes are permitted **only** during a SEV-1 or SEV-2 incident, made by the on-call administrator with the Incident Commander's approval recorded on the incident ticket. Within 24 hours a CHG ticket MUST record exactly what was changed; within 5 business days the change MUST be back-ported as a CL through the normal gates (or reverted).
3. Any other direct production change is a Flightpath violation. The owning team **owes a root-cause analysis (RCA)** within 5 business days, filed in Perch with the label `rca`, naming the change, who made it, why the process was bypassed, and the preventive action. The RCA is reviewed by the Office of the CTO; for changes touching security configuration, also by EntSec Governance.
4. A second violation by the same team within 12 months triggers a review of the team's production administrator access under KSS-IAM-02.
5. Accepting or using production administrator access that was not granted through the access-request process is itself a direct-production-change risk and is treated under this section: the recipient and the grantor both owe an RCA.

## 10. Adoption status by business unit

| Business unit / system | Roost | Gate 1 (Dev) | Gate 2 (UAT) | Gate 3 (Prod) | Notes |
|---|---|---|---|---|---|
| Kestrel corporate systems (CorpEng) | Full | Enforced | Enforced | Enforced | — |
| Flock, Watchtower (EntSec platforms) | Full | Enforced | Enforced | Enforced | — |
| **Kestrel Vantage: `vantage-salesforce`** | Onboarded 2022-09 | Enforced | Enforced | **Advisory until Q1 2027** | Per the Vantage integration plan: Gate 3 checks run and report; a Block, a missing CHG ticket or a second approval does **not** prevent merge to `main`. One code-owner approval suffices. Direct production changes remain violations under §9. |
| Kestrel Vantage: legacy applications (Birdseye) | Repository mirrored to Roost 2022-11 | Not enforced | Not enforced | Not enforced | Deprecated; no deploys through Flightpath. No code owner on file. |
| Plumage Health (Horizons) | Separate regime | — | — | — | Health-data controls; not covered here |

The Vantage adoption plan is tracked by Dana Whitfield (Director, Vantage Business Systems); the advisory period was set at acquisition integration planning and is reviewed at the Q4 2026 business-systems review.

## 11. Frequently asked

- **A Flow was changed in Setup in production to fix a routing bug. Is that fine?** No. It is a direct production change (§9). Back-port it as a CL and file the RCA.
- **The CL is merged. Can I close the ticket?** Only if the merge was to `main`, the deploy succeeded and verification is recorded. Otherwise update the ticket with the status from §7.
- **The Reviewer blocked on a clause that does not apply to us.** The code owner may override with a written reason. Overrides are reported; "does not apply" is reviewed by the Office of the CTO.
- **We need the change in production today and UAT would take a week.** If it is not SEV-1/2 incident response, it waits. Expedited Gate 2 and Gate 3 (same checks, same-day CHG approval) are available on request to CorpEng Release Engineering.
