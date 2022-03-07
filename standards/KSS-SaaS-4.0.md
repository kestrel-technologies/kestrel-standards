# Kestrel Security Standards (KSS) v4.0 — SaaS Platform Controls

**Document:** KSS-SaaS-4.0 · **Owner:** Enterprise Security, Governance (Hannah Kowalczyk) · **Effective:** March 7, 2022 · **Supersedes:** KSS-SaaS-3.2 (2021) · **Review cycle:** annual · **Classification:** Kestrel Internal

## 0. Purpose and scope

This standard defines the minimum security controls for any Software-as-a-Service platform that stores or processes **Kestrel Restricted** data, including customer personal data, customer contract and pricing data, and employee personal data. It applies to every business unit, including acquired entities from the date of acquisition close, regardless of the platform's onboarding status to central Kestrel services.

Controls are stated as requirements (MUST / MUST NOT / SHOULD). "Platform instance" means one tenant, org, workspace or account of a SaaS product (e.g., one Salesforce org). Production and non-production instances holding Restricted data are in scope; non-production instances holding only synthetic data are out of scope for KSS-MON-07 and KSS-NET-05.

Exceptions require a Perch ticket in the `ENTSEC-EXC` queue with compensating controls and an expiry date, approved by the Director, EntSec Governance. Verbal or chat approvals are not exceptions.

## 1. Controls

### KSS-IAM-01 · Single Sign-On

1. All human users MUST authenticate to the platform instance through Kestrel SSO (SAML 2.0 federation with the Kestrel identity provider, provisioned by Flock).
2. Local (username/password) authentication MUST be disabled for all human users.
3. One **break-glass** account MAY retain local authentication. It MUST be vaulted in Strongbox, MUST NOT be used for routine work, and its use MUST generate an alert.
4. User identities MUST be provisioned and deprovisioned by Flock. Accounts for individuals who have left Kestrel MUST be deactivated within 24 hours of separation.

### KSS-IAM-02 · Privileged access

1. Privileged (administrator-equivalent) access MUST be limited to the minimum number of named individuals required to operate the platform, and in any case **no more than two (2)** per production platform instance.
2. "Administrator-equivalent" means any user able to change security configuration, manage users, or read and modify all data irrespective of sharing rules.
3. Privileged accounts MUST be named individual accounts. Shared or generic administrator accounts are prohibited.
4. Privileged access MUST be reviewed quarterly; the review MUST be evidenced in Perch.

### KSS-IAM-03 · Integration and service accounts

1. Each integration MUST use its own dedicated service account with a dedicated permission profile granting only the objects, fields and operations the integration requires.
2. Service accounts MUST NOT hold administrator-equivalent permissions (as defined in KSS-IAM-02) unless an approved exception is on file.
3. Service accounts MUST be API-only; interactive (user-interface) login MUST be disabled.
4. Service account credentials MUST be rotated at least every 90 days and MUST be stored in Strongbox.

### KSS-SEC-04 · Secrets management

1. Credentials, API keys, tokens and private keys MUST NOT be stored in source code, build configuration, platform configuration records (including custom settings and custom metadata), documentation, tickets, or logs.
2. Secrets MUST be stored in Strongbox or in the platform's approved secret facility (for Salesforce: Named Credentials with External Credentials).
3. Applications MUST NOT write secrets or session tokens to application or debug logs.

### KSS-NET-05 · Network restrictions

1. Access to the platform instance MUST be restricted to Kestrel corporate network ranges (office egress and VPN) for all user profiles, including administrators and service accounts.
2. Where the platform supports it, network restrictions MUST be enforced on every request, not only at login.
3. Service accounts MUST be restricted to the specific egress addresses of the systems that use them.
4. Exceptions for vendor or partner access MUST be documented per KSS §0.

### KSS-API-06 · API and connected-application governance

1. Third-party and internal applications connecting via OAuth MUST be explicitly approved by a platform administrator; users MUST NOT be able to self-authorize applications.
2. OAuth grants MUST use the authorization-code flow with PKCE or a server-to-server flow. Implicit (token-in-URL) grants are prohibited. Access tokens MUST NOT appear in URLs.
3. Refresh tokens MUST expire: no longer than 24 hours of inactivity and no longer than 7 days absolute, unless an exception is on file.
4. Connected applications MUST enforce the platform's network restrictions (no IP relaxation).
5. API access MUST be enabled only for profiles that require it.

### KSS-MON-07 · Event monitoring

1. The platform instance MUST stream authentication, administrative, API, data-export and report events to **Watchtower** within 15 minutes of occurrence.
2. Event retention in Watchtower MUST be at least 12 months.
3. Login forensics / anomaly detection features MUST be enabled where available.

### KSS-DAT-08 · Public and guest access

1. Unauthenticated (guest/public) users of any portal, site or community MUST NOT have read access to objects containing customer or employee personal data.
2. Sharing of records to guest users MUST be prohibited by default; any record-level sharing to guests MUST be explicitly approved and limited to public content types.
3. Authenticated external users (customers, partners) MUST only see records belonging to their own account.

## 2. Verification

Business units MUST be able to demonstrate compliance with each control on request. Evidence SHOULD be derived from the platform's configuration (exports, API queries, screenshots of settings) rather than from attestation. Where EntSec provides an automated assessment, its results take precedence over manual attestation.

## 3. Compliance states

| State | Meaning |
|---|---|
| **Compliant** | Every requirement of the control is met by configuration evidence. |
| **Partially compliant** | Some requirements met; a remediation item exists or is being created. |
| **Non-compliant** | One or more requirements not met with no approved exception. |
| **Unable to verify** | Evidence could not be obtained with available access. This is a finding in itself. |

## 4. Reference: Salesforce-specific guidance (KSS Appendix S, informative)

For Salesforce orgs, EntSec provides this non-exhaustive mapping to help owners self-assess. The mapping is **informative**, not a substitute for the requirements above.

**Internal user.** An active user whose profile's user license is **Salesforce** or **Salesforce Platform**. Portal and community users, the site guest user, Chatter Free users and Salesforce system users on other licenses (for example the Analytics Cloud "Integration User" and "Security User") are not internal users. When an assessment counts users for IAM-01, IAM-02 or IAM-03, it counts internal users.

| Control | Where to look in Salesforce |
|---|---|
| IAM-01 | Single Sign-On Settings; profile permission "Is Single Sign-On Enabled"; user Federation ID; Login History by Login Type |
| IAM-02 | Administrator-equivalent (KSS-IAM-02 §2) means a user whose profile **or** any assigned permission set grants any of Modify All Data, Manage Users, Author Apex or Customize Application, whatever the profile is called (custom profiles count as much as System Administrator) |
| IAM-03 | Service account profiles; "Password Never Expires"; permission sets; API Enabled; login hours/IP ranges |
| SEC-04 | Apex classes, custom settings, custom metadata, Named Credentials, debug logs |
| NET-05 | Profile Login IP Ranges on every login-enabled profile. Login-enabled profile: the standard System Administrator profile plus every custom profile on a Salesforce or Salesforce Platform license, that is, profiles that have or could have internal users. Standard profiles with no users (Standard User, Solution Manager, and so on) are excluded; Session Settings: "Enforce login IP ranges on every request," "Lock sessions to the IP address from which they originated" |
| API-06 | Connected Apps: OAuth policies (Permitted Users, IP Relaxation, Refresh Token Policy), OAuth scopes, flows in use |
| MON-07 | Event Monitoring (Event Log Files), Real-Time Event Monitoring, Transaction Security Policies, Login Forensics, Field Audit Trail; Watchtower collector configuration for the org |
| DAT-08 | Experience Cloud guest user profile object permissions; sharing rules and sharing sets granting to guest users; external org-wide defaults |
