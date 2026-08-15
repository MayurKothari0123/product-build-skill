> FlowBreaker · review · prd.md · 2026-08-15
> Status: complete

# Assumptions

Gaps filled to keep the analysis moving. **These are not facts (R3).** Each one
below is a place where the source document is silent and FlowBreaker guessed. Every
downstream artifact that depends on one carries the dependency, and they reappear in
the readiness report.

| ID | Assumption | Source | Status | Affects | Risk if wrong |
|---|---|---|---|---|---|
| ASSUMP-001 | Leave is requested in whole days; no half-day support | domain_convention | unvalidated | REQ-001, REQ-006, FLOW-001 | Data model rework; half-days are common and would change balance arithmetic → Q-008 |
| ASSUMP-002 | One approver per request, not a chain | inferred | unvalidated | FLOW-002, REQ-004 | If Q-005 resolves to two-stage, the whole approval flow and state machine is wrong |
| ASSUMP-003 | Employees have exactly one line manager | domain_convention | unvalidated | FLOW-002, REQ-003 | Matrix reporting or dotted lines would make "the manager" ambiguous and route requests nowhere |
| ASSUMP-004 | Team calendar is visible to the whole team, not just the manager | inferred | unvalidated | REQ-007, FLOW-004 | Directly determines the severity of the privacy issue in Q-004 |
| ASSUMP-005 | Notifications are email-only, per §6 | evidence-adjacent | unvalidated | REQ-005, REQ-012 | §6 says email; if the portal has in-app notifications, email-only is a deliberate choice nobody recorded |
| ASSUMP-006 | Balance is checked at submission time, not at approval time | inferred | unvalidated | REQ-002, REQ-006, EDGE-005 | If checked only at submission, two pending requests can each pass and jointly overdraw the balance |
| ASSUMP-007 | "Balances reset annually" means on a fixed calendar date, org-wide | inferred | unvalidated | REQ-009, EDGE-006 | Anniversary-based reset per employee is equally common and changes the reset logic entirely |
| ASSUMP-008 | Rejected requests are terminal — no resubmission or appeal flow | inferred | unvalidated | FLOW-002, FLOW-005 | An employee correcting a typo would have to file fresh, losing the thread |

## Assumptions the source document itself makes

Not gaps FlowBreaker filled — beliefs the document rests on without acknowledging.
These are findings about the PRD, not about the system.

| Assumption embedded in the source | Where | Why it's worth surfacing |
|---|---|---|
| Managers respond promptly to email | §6 | The entire notification model is email-push with no escalation. The current process fails partly *because* people don't respond promptly; nothing here changes that. |
| HR reconciliation is the bottleneck worth fixing | §1 | Plausible, but §8's success criteria include employee chasing — which this design doesn't clearly address. |
| Every employee has a manager in the system | §2 | Contractors, new joiners before manager assignment, and the CEO all break this silently. |
| Leave policy is uniform across the organisation | §4 | No mention of location, seniority, or contract type varying entitlement. In a distributed team this is rarely true. |

## Validation status

**8 of 8 assumptions are unvalidated.** None has been confirmed by a person, tested
against real usage, or supported by a document. The analysis below is usable, but
its foundations are guesses that a five-minute conversation would settle.
