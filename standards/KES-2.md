# Kestrel Engineering Standards (KES) v2 — Secure Development and Release

**Document:** KES-2.0 · **Owner:** Office of the CTO, Engineering Standards (Daniel Reyes) · **Effective:** February 10, 2020 · **Classification:** Kestrel Internal

## Scope

All first-party code and Salesforce metadata deployed to a Kestrel production environment.

## Clauses

| Clause | Requirement (summary) |
|---|---|
| KES-SEC-1 | No secrets in code or configuration. |
| KES-SEC-2 | No injection: bind variables, parameterized queries. |
| KES-SEC-3 | Apex classes declare their sharing mode; exposed methods enforce object and field permissions. |
| KES-SEC-4 | No unsafe DOM handling in web components. |
| KES-SEC-5 | No personal data or secrets in logs. |
| KES-DEV-1 | Every change is reviewed by a code owner. |
| KES-DEV-2 | Tests assert outcomes. |
| KES-DEV-3 | Infrastructure and configuration as code. |
| KES-DEV-4 | Apex test coverage of at least 75% per class from asserting tests. |
| KES-DEV-5 | Dependencies pinned and scanned. |

## Exceptions

KES-SEC exceptions follow the `ENTSEC-EXC` process; KES-DEV exceptions use `KES-EXC`.
