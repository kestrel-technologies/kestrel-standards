# Kestrel Engineering Standards (KES) v3 — Secure Development and Release

**Document:** KES-3.0 · **Owner:** Office of the CTO, Engineering Standards (Daniel Reyes, Director) · **Effective:** July 1, 2025 · **Supersedes:** KES-2.4 (2022) · **Review cycle:** annual · **Classification:** Kestrel Internal

## 0. Purpose and scope

1. This standard defines the minimum engineering requirements for any code or configuration that reaches a Kestrel production system. It applies to:
   - Kestrel-built services and tools, in any language (today primarily Python, Go and TypeScript);
   - infrastructure and deployment definitions;
   - Salesforce metadata: Apex, Lightning Web Components (LWC), Visualforce, Flows and other declarative automation, profiles, permission sets, sharing configuration, connected apps and site (portal) configuration.
2. It applies to every business unit and to acquired entities from the date of acquisition close, regardless of their onboarding status to Roost or Flightpath.
3. Relationship to the Kestrel Security Standards (KSS): KSS defines the required **state** of a platform (who can log in, what is monitored). KES defines how **changes** to code and configuration are made and verified. A change that would move a platform out of compliance with KSS MUST NOT pass Flightpath.
4. Requirements use MUST / MUST NOT / SHOULD as in RFC 2119. "CL" (changelist) is the Roost term for a pull request. "Code owner" is the team named in the repository's `CODEOWNERS` file for the changed path. "Reviewer" means the Flightpath Reviewer, the automated standards check described in §5.

## 1. Definitions

| Term | Meaning |
|---|---|
| Roost | Kestrel source control and code review. Every repository that produces a production artifact lives in Roost. |
| Flightpath | The release process: `develop` → `uat` → `main`, with a gate at each promotion. See *Flightpath: Release Process* on Charter. |
| CL | A changelist (pull request) in Roost. The unit of review, test and promotion. |
| Code owner | The team accountable for a path, declared in `CODEOWNERS`. At least one code owner MUST approve a CL touching that path. |
| Strongbox | Kestrel's approved secrets vault. For Salesforce, the approved facility is Named Credentials backed by External Credentials. |
| Exposed method | Any Apex entry point callable from a browser or an external caller: `@AuraEnabled`, `@RemoteAction`, `@RestResource`, `@InvocableMethod`, `webservice`, Visualforce controller actions, and any method reachable from an Experience Cloud (portal) site. |
| Trust boundary | Any point where data enters code from a user, a browser, a file, an external API, a record field, or a queue message. |

## 2. Security requirements

### KES-SEC-1 · No secrets in code or configuration

1. Credentials, API keys, OAuth client secrets, access or refresh tokens, session identifiers, private keys and signing keys MUST NOT appear in source code, test code, build or deployment configuration, container images, documentation, CL descriptions, tickets, chat messages or log output.
2. Services MUST read secrets at runtime from Strongbox (or from the deployment platform's secret facility that is fed from Strongbox) through an injected environment variable or a mounted file. Secrets MUST NOT be committed in `.env` files; `.env.example` files MUST contain placeholders only.
3. Salesforce callouts MUST authenticate through a Named Credential with an External Credential. Apex, custom settings, custom metadata, custom labels, static resources and Flow variables MUST NOT hold credentials or tokens, in plain text or in a reversible encoding.
4. A secret that is found committed is considered compromised. It MUST be rotated before the CL is merged, even if the CL removes it, and the finding MUST be recorded in Perch.
5. Repositories MUST enable secret scanning and MUST NOT disable or allowlist it for production code. Test fixtures that require secret-shaped strings MUST use strings that are visibly synthetic (for example, a `_test` suffix) and MUST be documented in the repository README.
6. Secrets MUST be rotated at least every 90 days, or immediately after any suspected exposure (see also KSS-IAM-03 §4).

### KES-SEC-2 · No injection

1. **Queries.** All SOQL, SOSL and SQL that includes a runtime value MUST use bind variables (Apex `:variable`; parameterized statements in every other language). Building a query string by concatenation or interpolation of a runtime value is prohibited, including with `Database.query()`, `Database.queryWithBinds()` when the bind map is itself built from unvalidated input, `String.format`, or an ORM "raw" interface.
2. `String.escapeSingleQuotes()` or an equivalent escaping function MUST NOT be used as a substitute for binding. It MAY be used only where the platform offers no bind mechanism (for example, dynamic field names), and only after the value is validated against an allowlist.
3. **Input validation.** Every value crossing a trust boundary MUST be validated for type, length, range and format before use. Validation MUST use allowlists where the set of valid values is known (field names, object names, sort directions, enumerations).
4. **Commands and paths.** Shell commands MUST NOT be built from runtime values. File paths derived from input MUST be resolved and checked against an allowed root.
5. **OAuth and session handling.**
   - Applications acting as OAuth clients MUST use the authorization-code grant with PKCE, or a server-to-server grant (client credentials, JWT bearer). The implicit (user-agent) grant is prohibited.
   - Every authorization request MUST carry a `state` value that is unpredictable, bound to the user's session, and verified on the callback. A callback without a matching `state` MUST be rejected.
   - Access tokens, refresh tokens and session identifiers MUST NOT be placed in URLs (query string or fragment), including by forwarding a fragment value to the server as a query parameter. They MUST be carried in an `Authorization` header or a cookie marked `Secure`, `HttpOnly` and `SameSite`.
   - Redirect URIs MUST be matched exactly against a registered allowlist.
   - Sessions MUST expire after at most 24 hours of inactivity and MUST be invalidated on logout and on credential rotation.
6. **Deserialization.** Untrusted input MUST NOT be deserialized into executable or polymorphic types.

### KES-SEC-3 · Object, field and record access enforcement in Apex

1. Apex classes MUST be declared `with sharing` (or `inherited sharing` where the class is a utility invoked from other classes). `without sharing` MUST NOT be used unless the class carries a justification comment in the form `// KES-SEC-3 exception: <reason>` immediately above the class declaration and the CL is approved by the code owner. Classes with no sharing keyword are treated as `without sharing` by the Reviewer and MUST be corrected.
2. Every SOQL query in an exposed method MUST enforce object and field-level permissions of the running user, by one of: `WITH USER_MODE`, `WITH SECURITY_ENFORCED`, or `Security.stripInaccessible(AccessType.READABLE, records)` applied before the records leave the method.
3. Every DML statement in an exposed method MUST enforce object and field-level permissions, by one of: `Database.insert(records, AccessLevel.USER_MODE)` (and the equivalent for update, upsert, delete), or `Security.stripInaccessible(AccessType.CREATABLE | UPDATABLE, ...)` before the DML.
4. Exposed methods MUST NOT return fields the running user cannot read, even when the field is not displayed. Returning whole records "for convenience" and filtering in the browser is prohibited.
5. Record-level access MUST come from the sharing model (org-wide defaults, sharing rules, sharing sets, teams), not from filters in code. Code MUST NOT reimplement sharing with `WHERE OwnerId = :UserInfo.getUserId()`-style clauses as its only control.
6. Experience Cloud (portal) guest user profiles MUST NOT be granted read access to objects containing customer or employee personal data (see KSS-DAT-08). Sharing rules that grant records to a guest user MUST be reviewed by EntSec Governance before deployment.
7. Flows and other declarative automation MUST run in user context unless the Flow description documents why system context is required and the code owner has approved it (see KES-DEV-6).

### KES-SEC-4 · No unsafe DOM or output handling in LWC and web code

1. Web code MUST render data through the framework's templating (LWC template binding, React JSX, server-side auto-escaping templates). Writing markup from a string is prohibited: `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `eval`, `new Function`, `setTimeout(string)`, and React `dangerouslySetInnerHTML`.
2. In LWC, `lwc:dom="manual"` MUST NOT be used to insert content derived from record data, user input or any external source. Where a third-party library requires manual DOM, the inserted content MUST be text (`textContent`) or MUST pass through an approved sanitizer, and the CL MUST carry a code-owner approval naming the library.
3. Visualforce MUST NOT set `escape="false"` on output of user-controlled or record-derived values.
4. Rich-text fields MUST be displayed with `lightning-formatted-rich-text` (LWC) or the platform's rich-text output component, never by injecting the field value as HTML.
5. Services that render HTML MUST set a Content Security Policy that disallows inline scripts, and MUST set `X-Content-Type-Options: nosniff`.
6. URLs built from input MUST be validated against an allowlist of schemes (`https:`) and, for redirects, against an allowlist of destinations.

### KES-SEC-5 · No personal data or secrets in logs

1. Logs, traces, metrics labels, error reports and debug output MUST NOT contain: credentials, tokens, session identifiers, authorization headers, or personal data (names together with contact details, email addresses, phone numbers, government identifiers, precise location, or free-text fields that may contain any of these).
2. In Apex, `System.debug()` of a record, a list of records, `JSON.serialize()` of records, an `HttpRequest` or an `HttpResponse` body is prohibited in code that reaches production. Debug statements MAY log record Ids, counts, status codes and timings.
3. In services, logging MUST be structured (key-value), MUST apply a redaction filter for known secret and personal-data field names, and MUST default to `INFO` or higher in production.
4. Exception handlers MUST NOT log the exception message when the message may echo user input (for example, query strings or request bodies) without redaction.
5. Log retention and access are governed by KSS-MON-07; this clause governs content.

## 3. Development requirements

### KES-DEV-1 · Code review is required

1. Every change to a production repository MUST be made through a CL in Roost and MUST be approved by at least one code owner for every changed path before merge. The author MUST NOT be the approving code owner.
2. Direct pushes to `develop`, `uat` and `main` are prohibited. Branch protection MUST enforce this; administrators MUST NOT bypass it except under the Flightpath emergency procedure.
3. A CL MUST be small enough to review: SHOULD change one concern, SHOULD be under 400 changed lines excluding generated files, and MUST link the Perch ticket it implements.
4. Review MUST cover correctness, test adequacy and the KES clauses in §2. Approval with the comment "LGTM" alone is not evidence of review; the approver SHOULD note what was checked.
5. The Flightpath Reviewer (§5) MUST have run on the final revision of the CL. A `Block` verdict MUST be resolved or overridden in writing by the code owner before merge.
6. Salesforce declarative changes (profiles, permission sets, Flows, sharing rules, connected apps, site settings) are code for the purposes of this clause and MUST be retrieved into the repository and reviewed as a CL. Changes made in Setup directly in a production org are governed by Flightpath §9 (emergency changes) and otherwise constitute a violation.

### KES-DEV-2 · Tests are required and must assert outcomes

1. Every CL that changes behavior MUST include or update automated tests that exercise the change.
2. A test MUST assert the expected outcome. Tests that contain no assertion, that assert a constant (`System.assert(true)`, `assert True`, `expect(true).toBe(true)`), or that only assert that no exception was thrown are not tests and MUST be rejected in review.
3. Production code MUST NOT branch on whether it is running under test (`Test.isRunningTest()`, `NODE_ENV === 'test'`, `if TESTING:`) to skip logic. Test doubles (mocks, stubs, fakes) MUST be injected through interfaces or the platform mock facilities (`Test.setMock`, `Test.createStub`).
4. Tests MUST be deterministic: no dependence on wall-clock time, external networks, record Ids, or org data. Apex tests MUST NOT use `@IsTest(SeeAllData=true)`.
5. For each permission or sharing check in code, there MUST be at least one negative test: a user without the permission does not receive the data or the operation fails with the expected error.
6. CI MUST run the full test suite on every CL and MUST fail the CL on any test failure.

### KES-DEV-3 · Infrastructure and configuration as code

1. Infrastructure (projects, services, networking, IAM bindings, schedules, alert policies) MUST be defined in version-controlled files in Roost and applied by an automated pipeline. Manual changes made in a cloud console MUST be reproduced in code within 5 business days or reverted.
2. Salesforce metadata MUST be stored in source format (SFDX) in Roost. The `main` branch MUST represent the production org; the pipeline MUST detect drift between `main` and the org at least weekly and open a Perch ticket for any difference.
3. Environment-specific values (URLs, org Ids, feature flags) MUST live in configuration files or variables that are reviewed like code, not in the code itself and not in a person's local setup.
4. Every repository MUST be deployable from a clean checkout by the documented pipeline with no undocumented manual step.

### KES-DEV-4 · Apex test coverage with meaningful assertions

1. Apex classes and triggers MUST have at least **85%** line coverage per class and org-wide at the time of promotion to `main`. The platform's 75% deployment minimum is not sufficient.
2. Coverage MUST come from tests that satisfy KES-DEV-2. Coverage produced by tests without assertions, by `Test.isRunningTest()` shortcuts, or by tests that call methods only to execute lines does not count, and the Reviewer will flag such tests regardless of the reported percentage.
3. Test data MUST be created in the test (directly or through a test data factory). Tests MUST NOT depend on org data.
4. Tests for exposed methods MUST include at least one test running as a non-administrator user (`System.runAs`) with a profile or permission set representative of the real users of the method.
5. Callouts MUST be tested with `HttpCalloutMock` or `WebServiceMock` implementations that return realistic responses, including at least one error response.

### KES-DEV-5 · Dependency hygiene

1. Every repository MUST declare its dependencies in a manifest with a lockfile (`requirements.txt` with pinned versions or `poetry.lock`/`uv.lock`; `go.sum`; `package-lock.json`/`pnpm-lock.yaml`), committed to Roost.
2. Dependencies MUST be pinned to an exact version. Floating ranges are prohibited in production manifests.
3. CI MUST run a vulnerability scan against the lockfile on every CL and at least weekly on `main`. A dependency with a known critical or high vulnerability MUST be upgraded or replaced before promotion to `main`, or an exception MUST be on file (§6).
4. Dependencies MUST be fetched from the Kestrel-approved registries or mirrors. Vendored copies of third-party code are prohibited unless the code owner documents why and takes ownership of patching.
5. Unused dependencies MUST be removed. Dependencies MUST have an acceptable open-source license per the Legal allowlist.
6. Salesforce managed packages are dependencies: installed versions MUST be recorded in the repository and reviewed for upgrades at least quarterly.

### KES-DEV-6 · Flows and declarative automation are documented and tested

1. Every Flow, Process, Workflow Rule, validation rule, approval process and assignment rule MUST have a Description stating its purpose, trigger, the objects it reads and writes, and its run mode (user or system context).
2. Every active Flow MUST be listed in the repository's `docs/flows.md` with owner, trigger, and a link to its test.
3. Every record-triggered or scheduled Flow MUST have an automated test (a Flow test or an Apex test that triggers it) with at least one assertion on the resulting record state, including one case where the Flow should **not** act.
4. Flows MUST run in user context (`Default` run mode). `System Context Without Sharing` MUST NOT be used unless the Description states the reason and the code owner approved the CL (see KES-SEC-3 §7).
5. Field defaulting and record-type or picklist assignment performed by automation MUST be covered by a test that confirms the default is applied only in the intended condition. Automation that sets fields used in sharing rules or in portal visibility is high-risk and MUST be reviewed by EntSec Governance.
6. Inactive Flow versions older than the two most recent MUST be deleted. A Flow that no longer has a documented purpose MUST be deactivated.

## 4. Dependency requirements

### KES-DEP-1 · No end-of-life dependencies

1. A runtime, framework, library or platform API version that has passed its vendor's end-of-life date, or that has received no security release for 12 months while newer major versions exist, MUST NOT be used in production.
2. Minimum versions as of this revision (updated by the Office of the CTO without a new KES revision; the current table is on Charter):

| Technology | Prohibited in production | Minimum |
|---|---|---|
| Python | 3.9 and earlier | 3.10 |
| Flask | 1.x | 2.3 |
| requests (Python) | earlier than 2.31 | 2.31 |
| Jinja2 | 2.x | 3.1 |
| Node.js | 18 and earlier | 20 LTS |
| Go | earlier than 1.21 | 1.21 |
| Salesforce API version | more than three releases behind the org's current version | current − 3 |

3. CI MUST fail a CL that introduces or retains a prohibited version. An exception (§6) MUST include a decommission or upgrade date no more than 6 months out.
4. Applications that cannot be upgraded MUST be scheduled for decommission, recorded in the Internal Applications Catalog as **Deprecated** with a target date, and MUST NOT receive new features.

## 5. How Flightpath checks this

The **Flightpath Reviewer** is an automated reviewer that runs on every CL in Roost, in addition to the repository's own CI (build, tests, validation deploy). It reads the CL diff and the full text of the changed files, evaluates them against the clauses in this document, and posts one review comment with a verdict table:

| Column | Content |
|---|---|
| Clause | KES-SEC-1 … KES-DEP-1 |
| Verdict | **Pass** (no finding) · **Warn** (likely finding; human judgment required) · **Block** (clear violation; merge is prevented until resolved or overridden) · **n/a** (clause not applicable to the changed files) |
| Finding | File path, line, and a one-sentence statement of the violation |
| Fix | The specific change that would satisfy the clause |

Mapping of clauses to checks:

| Clause | Automated (Reviewer and CI) | Requires a human reviewer |
|---|---|---|
| KES-SEC-1 | Secret patterns in diff; credential-shaped literals in Apex, custom settings, custom metadata, YAML/JSON config; secret scanning | Whether a Named Credential or Strongbox entry is correctly configured |
| KES-SEC-2 | Concatenated `Database.query`/SQL; missing `state` on OAuth authorize URLs; `response_type=token`; tokens in URL construction; shell/`exec` with runtime values | Adequacy of input validation |
| KES-SEC-3 | Missing sharing keyword; `without sharing` without justification comment; SOQL/DML in exposed methods lacking `USER_MODE`, `SECURITY_ENFORCED` or `stripInaccessible`; guest profile object permissions in metadata | Business justification for exceptions; sharing-model design |
| KES-SEC-4 | `innerHTML` and the prohibited list; `lwc:dom="manual"`; `escape="false"`; `dangerouslySetInnerHTML` | Whether a sanitizer is adequate |
| KES-SEC-5 | `System.debug` of records/serialized objects; log statements containing `token`, `password`, `secret`, `Authorization`, email-shaped or phone-shaped values | Redaction completeness |
| KES-DEV-1 | Branch protection; code-owner approval; Reviewer ran on final revision | Quality of review |
| KES-DEV-2 | Assertion-free tests; constant assertions; `Test.isRunningTest()` branches in non-test code; `SeeAllData=true` | Whether assertions test the right thing |
| KES-DEV-3 | Drift report (weekly job); unmanaged files | Whether the pipeline is the only path to production |
| KES-DEV-4 | Per-class and org-wide coverage from the validation deploy; `runAs` presence; callout mocks | Meaningfulness of coverage |
| KES-DEV-5 | Lockfile present; unpinned ranges; vulnerability scan | Replacement choices |
| KES-DEV-6 | Flow Description present; run mode; entry in `docs/flows.md`; Flow test exists | Whether the documentation is accurate |
| KES-DEP-1 | Manifest versions against the EOL table | Decommission plans |

Rules of use:

1. The Reviewer is a reviewer, not an approver. A Pass does not substitute for KES-DEV-1.
2. A Block MAY be overridden only by a code owner, with a comment on the CL in the form `flightpath: override <clause> — <reason>`. Overrides are reported weekly to the Office of the CTO and to EntSec Governance for security clauses.
3. Reviewer verdicts are advisory where a business unit's Flightpath adoption plan says so (see Flightpath §10). Advisory status changes what blocks a merge; it does not change what this standard requires.

## 6. Exceptions

1. Exceptions to KES-SEC clauses are security exceptions and follow KSS §0: a Perch ticket in the `ENTSEC-EXC` queue with compensating controls and an expiry date, approved by the Director, EntSec Governance.
2. Exceptions to KES-DEV and KES-DEP clauses require a Perch ticket in the `KES-EXC` queue approved by the Director, Engineering Standards (Office of the CTO), with an expiry date no more than 6 months out.
3. Architecture reviews that identify KES findings are recorded as `KSS-REV` tickets; a review finding accepted by a business owner is not an exception and does not stop the clock on remediation.
4. Chat or verbal approvals are not exceptions.

## 8. Pre-CL checklist (informative)

Authors SHOULD confirm each line before requesting review. Reviewers MAY ask for the completed list.

- [ ] No credential, key, token or session identifier in the diff, in test data, or in the CL description (KES-SEC-1).
- [ ] Every query with a runtime value uses a bind; every OAuth request carries `state`; no token appears in a URL (KES-SEC-2).
- [ ] Every Apex class states its sharing mode; every exposed method enforces object and field-level permissions on read and write (KES-SEC-3).
- [ ] No `innerHTML`, `lwc:dom="manual"` with data, `escape="false"` or equivalent (KES-SEC-4).
- [ ] No debug or log statement emits a record, a serialized object, or a value that could be personal data or a secret (KES-SEC-5).
- [ ] CL links a Perch ticket; a code owner is requested; the author is not the approver (KES-DEV-1).
- [ ] Tests exist, assert outcomes, include a negative permission case, and do not branch on test mode (KES-DEV-2).
- [ ] Infrastructure or declarative configuration changed in a console or Setup is reproduced in this CL (KES-DEV-3).
- [ ] Apex per-class coverage ≥ 85% from asserting tests; callouts mocked (KES-DEV-4).
- [ ] Dependencies pinned and locked; vulnerability scan green (KES-DEV-5); no version in the EOL table (KES-DEP-1).
- [ ] Every changed or new Flow has a Description, an entry in `docs/flows.md`, a test, and runs in user context or documents why not (KES-DEV-6).
- [ ] Flightpath Reviewer has run on the final revision; every Warn is answered in a comment.

## 9. Revision history

| Version | Date | Change |
|---|---|---|
| 3.0 | 2025-07-01 | Added KES-SEC-3 §2–3 (`USER_MODE`), KES-DEV-6, KES-DEP-1; raised Apex coverage floor to 85%; introduced the Flightpath Reviewer verdict table |
| 2.4 | 2022-10-03 | OAuth requirements (KES-SEC-2 §5) made explicit after acquisition reviews |
| 2.0 | 2020-02-10 | First version covering Salesforce metadata |
