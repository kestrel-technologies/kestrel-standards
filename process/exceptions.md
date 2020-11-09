# Exceptions to Kestrel Standards

**Owners:** Enterprise Security, Governance (security exceptions) · Office of the CTO, Engineering Standards (engineering exceptions) · **Classification:** Kestrel Internal

An exception is a written, time-limited permission not to meet part of a standard, with compensating controls. It exists only as an approved Perch ticket.

## 1. Which queue

| You need an exception to | Perch queue | Approver | Maximum duration |
|---|---|---|---|
| Any KSS control, or a KES-SEC clause | `ENTSEC-EXC` | Director, EntSec Governance | Set by the approver; an expiry date is mandatory |
| A KES-DEV or KES-DEP clause | `KES-EXC` | Director, Engineering Standards (Office of the CTO) | 6 months |

## 2. What the request must contain

1. **Scope:** the platform instance (for example, one Salesforce org), the control or clause, and exactly which requirement is not met.
2. **Reason** the requirement cannot be met today.
3. **Compensating controls** in place for the duration.
4. **Expiry date**, and the plan that removes the need for the exception by then.
5. **Owner** accountable for the plan.

## 3. Rules

1. **Chat approvals are not exceptions.** Neither are verbal approvals, emails, meeting notes or a manager's agreement. If there is no approved ticket, there is no exception.
2. An exception covers only the scope written on the ticket. It does not extend to other instances, controls or teams.
3. An exception ends on its expiry date. From that day the control applies in full. Renewal is a new request, filed before expiry and approved like the first.
4. An architecture-review finding (`KSS-REV`) that a business owner accepts as risk is not an exception and does not stop the remediation clock.
5. An incident does not create an exception. Acquired entities are in scope from the date the acquisition closes (KSS §0, KES §0) unless an approved exception says otherwise.

