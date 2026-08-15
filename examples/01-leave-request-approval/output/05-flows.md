> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# User Flow Audit

Five flows. Four are derivable from the source; **FLOW-005 is absent from it
entirely** and is included because users will attempt it on day one.

FLOW-003 (HR review) was deleted after Q-005 confirmed §5 is stale.

---

## FLOW-001 — Employee submits a leave request

**Actor:** Employee · **Trigger:** Employee needs time off
**Requirements:** REQ-001, REQ-002, REQ-008, REQ-012, REQ-013
**Happy path:** Open form → pick type, dates → see balance impact → submit → confirmation → manager notified

### States

| ID | Name | Type | Trigger | Expected behaviour | Recovery | Eval? |
|---|---|---|---|---|---|---|
| STATE-001 | First-time, no history | empty | Employee opens the module, has never requested | Show balance and an explanation of the process, not an empty table | n/a | yes |
| STATE-002 | Balance loading | loading | Form opens | Disable submit until the balance is known; do not show a stale or zero balance | Retry; on failure block submission with a reason | yes |
| STATE-003 | Balance unavailable | error | Balance service fails | **UNSPECIFIED.** Submit-anyway-and-validate-later, or block? | unknown | yes |
| STATE-004 | Insufficient balance | error | Requested days > balance, type = annual | Block with the specific numbers: requested vs available | Adjust dates or switch type | yes |
| STATE-005 | Submitted, pending | success | Submit succeeds | Confirm, show status, state who decides and by when | n/a | yes |
| STATE-006 | Submit failed | error | Network or server failure on submit | **UNSPECIFIED.** Is the form preserved? | unknown | yes |

### Missing and broken paths

| Finding | Impact | Ref |
|---|---|---|
| No behaviour for a failed balance lookup | Either blocks all leave during a partial outage, or lets people overdraw silently | STATE-003, EDGE-008 |
| No statement that form input survives a failed submit | Users retype everything; on mobile many just give up | STATE-006, EDGE-009 |
| Balance checked at submission only *(ASSUMP-006)* | Two pending requests can each pass the check and jointly overdraw | EDGE-005, TEST-006 |
| No overlap check against the employee's own existing leave | Duplicate/overlapping requests silently accepted | EDGE-010 |

---

## FLOW-002 — Manager decides a request

**Actor:** Manager (or skip-level, for a manager's own request)
**Trigger:** Email notification of a pending request *(§6)*
**Requirements:** REQ-003, REQ-004, REQ-005, REQ-006, REQ-010, REQ-015
**Happy path:** Open notification → see request → approve or reject with reason → employee notified → balance deducted

### States

| ID | Name | Type | Trigger | Expected behaviour | Recovery | Eval? |
|---|---|---|---|---|---|---|
| STATE-007 | No pending requests | empty | Manager opens the queue, nothing waiting | Explain that requests appear here; link to team calendar | n/a | yes |
| STATE-008 | Queue loading | loading | Queue opens | Skeleton rows; no action until loaded | Retry | yes |
| STATE-009 | Approver deactivated | permission | Manager left while request pending | Auto-reassign to skip-level; surface in HR exception view *(Q-002)* | Automatic | yes |
| STATE-010 | Manager's own request | permission | Manager submits their own leave | Route to skip-level; do not show in own queue *(Q-001)* | n/a | yes |
| STATE-011 | No skip-level exists | error | CEO or direct report of CEO submits | **UNSPECIFIED** → Q-015 | unknown | yes |
| STATE-012 | Already decided | concurrent | Two approvers open the same request; one decides first | Second sees the decision and who made it — not a stale action button | Refresh to current state | yes |
| STATE-013 | Decided after employee cancelled | concurrent | Employee cancels while manager is deciding | **UNSPECIFIED** — depends on Q-009 | unknown | yes |
| STATE-014 | Rejection reason empty | error | Manager rejects without a reason | Block; REQ-010 requires a reason | Enter reason | yes |
| STATE-015 | Decision failed to save | error | Server failure on submit | **UNSPECIFIED.** Manager may believe they decided when they didn't | unknown | yes |

### Missing and broken paths

| Finding | Impact | Ref |
|---|---|---|
| Browser back after deciding re-shows the pending screen | Manager may approve twice; without idempotency the balance is deducted twice | EDGE-007, TEST-012 |
| Two managers can decide concurrently *(ASSUMP-003 — assumes one manager)* | Conflicting decisions; employee gets two contradictory emails | STATE-012, EDGE-011 |
| No team-coverage view when deciding | Manager leaves the tool to check the calendar — the informal process this replaces | Q-007 |
| No escalation if the manager never acts | Requests sit pending indefinitely; the original complaint, rebuilt | EDGE-012 |

### Not applicable
- **Partial-data state:** N/A — the queue is a single small query. It loads or it fails.

---

## FLOW-004 — Team views the calendar

**Actor:** Any employee · **Requirements:** REQ-007
**Happy path:** Open calendar → see who is away

| ID | Name | Type | Expected behaviour | Eval? |
|---|---|---|---|---|
| STATE-016 | Nobody away | empty | Show the period with an explicit "no leave booked" | yes |
| STATE-017 | Calendar loading | loading | Skeleton; do not show an empty calendar that looks like "nobody away" | yes |
| STATE-018 | Viewer is a peer | permission | Show "unavailable" only; never the leave type *(Q-004)* | yes |
| STATE-019 | Viewer is the manager | permission | Show type for direct reports only | yes |

**Note.** STATE-017 is the sharpest failure in this flow and the easiest to get
wrong: an empty calendar and a loading calendar look identical, and the wrong read
("nobody is away, I'll book my leave then") is silent and consequential.

---

## FLOW-005 — Employee cancels or amends a request

**Not in the source document.** Included because it is inevitable: plans change,
dates get typo'd, illness resolves. With no flow, users will delete-and-resubmit or
email HR — recreating the manual process the project exists to eliminate.

**Entirely blocked on Q-009.** States below are the shape of the answer, not the
answer.

| ID | Name | Type | Question |
|---|---|---|---|
| STATE-020 | Cancel before decision | — | Silent withdrawal, or is the manager notified? |
| STATE-021 | Cancel after approval | — | Does the balance restore? Automatically? |
| STATE-022 | Amend after approval | — | New request, or re-approval of the existing one? |
| STATE-023 | Cancel after the leave started | — | Partial restoration? Any restoration? |

---

## Flow coverage summary

| Flow | Empty | Loading | Error | Success | Permission | Concurrent |
|---|---|---|---|---|---|---|
| FLOW-001 | ✓ | ✓ | ✓ (2 unspecified) | ✓ | — | ✗ |
| FLOW-002 | ✓ | ✓ | ✓ (2 unspecified) | ✓ | ✓ | ✓ |
| FLOW-004 | ✓ | ✓ | ✗ | ✓ | ✓ | n/a |
| FLOW-005 | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |

**3 of 4 flows have empty, loading and error states defined or explicitly N/A.**
FLOW-005 has none because it does not exist in the source. FLOW-004 has no error
state — what a viewer sees when the calendar fails to load is unaddressed, and given
STATE-017's failure mode, that gap matters more than its severity suggests.
