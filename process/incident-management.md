# Incident Management

**Owner:** Enterprise Security, Detection & Monitoring (incident response) · **Applies to:** every Kestrel business unit, including acquired entities · **Classification:** Kestrel Internal

## 1. What counts as an incident

An incident is any event that degrades a production system or exposes, or may have exposed, Kestrel data to someone who should not have it. If you are not sure, open the ticket; Incident Response will downgrade it.

## 2. Severity

| Severity | Meaning | Examples |
|---|---|---|
| **SEV-1** | Critical. Confirmed exposure of regulated data (payment, health or government identifiers), or a production outage of a revenue-critical system. | Regulated customer data published; production org unavailable to all users |
| **SEV-2** | Major. Exposure of customer or employee data with no regulated data categories confirmed, or a production outage of one business system. | Customer contact data viewable by outsiders through a shared link |
| **SEV-3** | Minor. Degraded service or a contained security weakness with no evidence of data exposure. | A scheduled job failing; a misconfiguration found before use |

Severity can change as facts arrive. Record each change and its reason on the ticket.

## 3. Roles

- **Incident Commander (IC).** Runs the response, approves containment steps and any direct production change (Flightpath §9). For security incidents the IC is the EntSec Incident Response on-call.
- **Business owner.** The director accountable for the affected system. Owns customer impact, the open actions and the root-cause analysis.
- **Responders.** Administrators and engineers the IC pulls in. They record what they change, when and why on the ticket.

## 4. How an incident runs

1. **Open** an `INC-####` ticket in Perch with the labels `incident` and the severity (`sev1`, `sev2`). Page EntSec Incident Response for any suspected data exposure.
2. **Contain.** Stop the exposure first: remove the link, revoke the tokens, disable the account. A deploy-related SEV-1 or SEV-2 needs a rollback-or-fix decision within 30 minutes (Flightpath §8).
3. **Communicate.** The IC posts status on the ticket. Customer notifications go out through the business owner and Legal, never directly from responders.
4. **Recover** and confirm the system is back to its intended state.
5. **Analyse.** Every SEV-1 and SEV-2 needs a root-cause analysis (RCA), filed in Perch with the label `rca`. Until it is filed the ticket status reads "Contained; RCA pending".
6. **Close** when the RCA is accepted and every open action has an owner and a due date.

## 5. What a root-cause analysis contains

- What happened, with a timeline in Pacific Time.
- The root cause or causes, and the contributing factors. "Human error" is never a root cause on its own: say which control should have caught it and why it did not.
- The controls involved, by KSS or KES identifier, and whether each was in place.
- Corrective and preventive actions, each with an owner and a due date, tracked as Perch tickets.

Where the cause depends on a platform's control posture, the business owner may commission an assessment against KSS first; the RCA cites its results.

## 6. Related

- [Flightpath](flightpath.md) §8 (rollback) and §9 (emergency and direct production changes).
- [Exceptions](exceptions.md): an incident does not create an exception.
- [Recent incidents](../incidents/README.md).
