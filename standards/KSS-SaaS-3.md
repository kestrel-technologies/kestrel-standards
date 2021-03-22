# Kestrel Security Standards (KSS) v3.1 — SaaS Platform Controls

**Document:** KSS-SaaS-3.1 · **Owner:** Enterprise Security, Governance (Hannah Kowalczyk) · **Effective:** April 1, 2021 · **Classification:** Kestrel Internal

## Scope

Every Software-as-a-Service platform instance that stores Kestrel Restricted data (customer personal
data, contracts, pricing, employee personal data). Acquired entities are in scope from the date the acquisition closes.

## Controls

| Control | Requirement (summary) |
|---|---|
| KSS-IAM-01 | Users sign in through the Kestrel identity provider; local passwords disabled except break-glass. |
| KSS-IAM-02 | Administrator access limited to named individuals and reviewed quarterly. |
| KSS-IAM-03 | Integration accounts are dedicated, least-privileged and rotated at least annually. |
| KSS-SEC-04 | No credentials in code or configuration; secrets live in Strongbox. |
| KSS-NET-05 | Logins restricted to Kestrel network ranges where the platform supports it. |
| KSS-API-06 | Connected applications and API clients are approved, scoped and reviewed. |
| KSS-MON-07 | Authentication and administrative events sent to central monitoring. |
| KSS-DAT-08 | No guest or public access to customer data. |

## Exceptions

Exceptions require a Perch ticket in the `ENTSEC-EXC` queue, approved by the Director, EntSec Governance.
