> FlowBreaker · review · prd.md · 2026-08-15
> Status: complete

# PRD Audit

## Requirements

| ID | Requirement | Clarity | Basis | Flows | Tests | Open Qs |
|---|---|---|---|---|---|---|
| REQ-001 | Employee submits a request with start date, end date, leave type | ambiguous | evidence §3 | FLOW-001 | TEST-001, TEST-009 | Q-006, Q-008 |
| REQ-002 | System shows the employee their remaining balance | ambiguous | evidence §3 | FLOW-001 | TEST-002 | Q-006, Q-011 |
| REQ-003 | Managers see pending requests for their team | clear | evidence §3 | FLOW-002 | TEST-003 | — |
| REQ-004 | Manager can approve or reject; rejection requires a reason | contradictory | evidence §3, §5 | FLOW-002 | TEST-004, TEST-010 | **Q-005**, Q-001, Q-007 |
| REQ-005 | Employee is notified of the decision | ambiguous | evidence §3 | FLOW-002 | TEST-005 | Q-013 |
| REQ-006 | Approved leave is deducted from the balance | ambiguous | evidence §3 | FLOW-002 | TEST-006 | Q-006, Q-008 |
| REQ-007 | Approved leave appears on the team calendar | ambiguous | evidence §3 | FLOW-004 | TEST-007 | **Q-004** |
| REQ-008 | Managers notified by email on submission | clear | evidence §6 | FLOW-001 | TEST-008 | Q-013 |
| REQ-009 | Leave balances reset annually | ambiguous | evidence §3 | **none** | — | Q-010 |
| REQ-010 | Rejection requires a reason | clear | evidence §3 | FLOW-002 | TEST-004 | — |
| REQ-011 | HR can export a monthly report of all leave taken | ambiguous | evidence §3 | **none** | — | Q-012 |
| REQ-012 | Sick leave requires no balance | clear | evidence §4 | FLOW-001 | — | Q-011 |
| REQ-013 | Unpaid leave does not affect the balance | clear | evidence §4 | FLOW-001 | — | — |
| REQ-014 | "The approval process should be fast" | **missing** | evidence §3 | — | — | Q-003 |

`clarity`: `clear|ambiguous|contradictory|missing`

---

## Contradictions

### C-1 — Who approves? *(critical)*
> **§2:** "The employee's line manager approves or rejects it."
> **§5:** "HR reviews and approves all leave requests to ensure policy compliance
> before the leave is confirmed."

These cannot both describe the sole approver. Three readings are possible and the
document supports none of them over the others:

1. Two-stage approval that is never described as two-stage anywhere else — §3, §6
   and §8 all read as single-stage.
2. §5 is stale text from an earlier draft.
3. §2 is aspirational and §5 is current practice.

**This is the single most consequential defect in the document.** Approval flow,
permission model, notification design, and the "should be fast" goal all depend on
it. → **Q-005**

### C-2 — Speed vs. an added review step *(medium)*
§8 targets employees no longer chasing, and §3 asks for a fast approval process.
§5 adds an HR review step to every request. Adding a serial approver to a process
whose complaint is latency is a direction, not a contradiction — but it is
undiscussed, which suggests nobody noticed. → Q-003, Q-005

---

## Undefined terminology

| Term | Used in | Why it matters |
|---|---|---|
| "working day" | §3, implied throughout | Are weekends excluded? Public holidays? Whose, for a distributed team? Every balance calculation depends on it. → Q-006 |
| "balance" | §3, §4 | Days or hours? Accrued monthly or granted annually? Can it go negative? → Q-006, Q-011 |
| "team" | §3 | Direct reports only, or the whole department? Determines both the calendar's audience and the manager's queue. → Q-004 |
| "fast" | §3 | Unmeasurable as written. → REQ-014 |
| "annually" | §3 | Calendar year, fiscal year, or employment anniversary? → Q-010 |
| "confirmed" | §5 | A fourth state alongside pending/approved/rejected, or a synonym for approved? The state machine differs. → Q-005 |

---

## Unfalsifiable requirements

| Stated | Problem | Suggested |
|---|---|---|
| "The approval process should be fast" (§3) | No subject, no threshold, no measurement point | "90% of requests receive a decision within 2 working days of submission" |
| "Intuitive and easy to use" (§8) | Not a criterion; cannot pass or fail | "A new employee completes a first request without help, in under 90 seconds, in usability testing (n≥5)" |
| "Employees stop chasing managers" (§8) | Real outcome, but unmeasured and untraceable to any system behaviour | "Median time-to-decision under 2 working days" plus "request status is visible to the employee at all times" |
| "HR no longer reconciles leave manually" (§8) | The only genuinely testable criterion here | Keep. Add the baseline: currently ~2 days/month. |

---

## Silent scope

Things the document implies without stating, each of which a reader could
reasonably assume is in or out:

- **A cancellation or amendment flow.** Requests can be submitted and decided;
  nothing describes changing or withdrawing one. Users will expect it. → Q-009
- **Half-days.** Ubiquitous in practice, absent here. → Q-008 / ASSUMP-001
- **An approval history or audit trail.** §5 invokes "policy compliance" but nothing
  records who decided what, when. Compliance without a trail is unenforceable. → Q-014
- **A manager's own leave.** Managers are employees; the document only casts them as
  approvers. → Q-001
- **Delegation while an approver is away.** An approver on leave is exactly the
  situation this system will encounter constantly. → Q-002

---

## Requirements quality summary

- **Total 14** — clear 5 · ambiguous 7 · contradictory 1 · missing 1
- **Requirements with no acceptance criteria: 14 of 14.** The document contains none.
- **Requirements with no linked flow: 2** — REQ-009 (balance reset), REQ-011 (HR export)
- **Requirements resting on ≥1 unvalidated assumption: 9**
- **Non-goals stated: 1.** Only payroll integration is excluded.

The document is a clear and readable *proposal*. It is not yet a specification: it
describes what the feature is, and almost never what the system should do when
something is unusual, absent, concurrent, or wrong.
