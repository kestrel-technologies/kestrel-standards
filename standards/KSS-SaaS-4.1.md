# Kestrel Security Standards (KSS) v4.1 — SaaS Platform Controls (superseded)

**Document:** KSS-SaaS-4.1 · **Owner:** Enterprise Security, Governance (Hannah Kowalczyk) · **Effective:** 2024 · **Status:** Superseded on January 15, 2026 by [KSS-SaaS-4.2](KSS-SaaS-4.2.md) · **Classification:** Kestrel Internal

> This version is kept for reference only. Every assessment from January 15, 2026 onward uses [KSS-SaaS-4.2](KSS-SaaS-4.2.md). An assessment that cites 4.1 is out of date.

## Changes in 4.2

| Control | KSS-SaaS-4.1 | KSS-SaaS-4.2 |
|---|---|---|
| KSS-MON-07 · Event monitoring | Streaming to Watchtower within 15 minutes, 12 months of retention, login forensics where available. No requirement for real-time policy enforcement. | Adds §3: where the platform supports real-time policy enforcement (for example, Salesforce Transaction Security), policies MUST block or alert on bulk data export and on administrative actions from unrecognized networks. Login forensics moves to §4. |
| All other controls | — | Editorial changes only. Control IDs are unchanged. |

## Where to find the full text

The full 4.1 text is in the history of this repository. Read [KSS-SaaS-4.2](KSS-SaaS-4.2.md) for the current requirements.
