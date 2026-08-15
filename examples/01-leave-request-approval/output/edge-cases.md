> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Edge Case Register

| ID | Edge case | Category | Likelihood | Impact | Specified? | Flows | Test |
|---|---|---|---|---|---|---|---|
| EDGE-001 | Request submitted for a past date | input | high | medium | no | FLOW-001 | TEST-009 |
| EDGE-002 | Request spans a weekend or public holiday | data | high | high | no | FLOW-001 | TEST-013 |
| EDGE-003 | Request spans the annual reset date | timing | medium | high | no | FLOW-001 | TEST-014 |
| EDGE-004 | Approver deactivated while request pending | permission | medium | critical | **yes** *(Q-002)* | FLOW-002 | TEST-011 |
| EDGE-005 | Two pending requests jointly exceed the balance | data | high | high | no | FLOW-001 | TEST-006 |
| EDGE-006 | Annual reset runs while requests are pending | timing | medium | high | no | — | TEST-014 |
| EDGE-007 | Manager approves twice via browser back | timing | medium | high | no | FLOW-002 | TEST-012 |
| EDGE-008 | Balance service down at submission time | network | medium | medium | no | FLOW-001 | TEST-015 |
| EDGE-009 | Submit fails; form contents lost | network | medium | medium | no | FLOW-001 | TEST-016 |
| EDGE-010 | Employee submits overlapping requests | input | medium | medium | no | FLOW-001 | TEST-017 |
| EDGE-011 | Two approvers decide the same request concurrently | timing | low | high | no | FLOW-002 | TEST-018 |
| EDGE-012 | Manager never acts on a request | timing | high | high | no | FLOW-002 | TEST-019 |
| EDGE-013 | Employee with no manager assigned submits | permission | medium | high | no | FLOW-001 | TEST-020 |
| EDGE-014 | Employee leaves with approved future leave | data | medium | medium | no | — | — |
| EDGE-015 | Sick leave submitted retroactively for a long absence | input | high | medium | no | FLOW-001 | TEST-009 |
| EDGE-016 | Notification email bounces or is filtered | network | high | high | no | FLOW-001, FLOW-002 | TEST-021 |

`category`: `data|timing|permission|network|input|scale`

**16 edge cases · 1 currently specified.** The one that is specified became so
because Q-002 was answered — it did not start that way.

---

## The ones worth reading

### EDGE-012 — Manager never acts *(likelihood high, impact high)*
**Description:** A request is submitted and the manager simply doesn't decide.
**Why it matters:** This is the *original problem*. §1 describes leave being handled
informally over WhatsApp; §8 wants employees to stop chasing. Building a system with
no escalation, no reminder, and no SLA reproduces the failure in a new interface —
except now the employee is chasing through a tool that gave them no way to chase.
**Currently specified:** no. §6 sends one email on submission and nothing after.
**Confidence:** inference — the source defines notification but never follow-up.
**Suggested handling:** Reminder at N days, escalation to skip-level at M days,
visible "waiting X days" status for the employee. This is a product decision, and it
is closer to the point of the project than most of the stated requirements.

### EDGE-002 — Weekends and public holidays *(likelihood high, impact high)*
**Description:** Leave from Friday to Monday. Is that 4 days or 2?
**Why it matters:** Every balance calculation depends on the answer, and there is no
answer. For a distributed team it is worse: whose public holidays? An employee in
one country loses days another doesn't.
**Currently specified:** no — "working day" is never defined. → Q-006
**Confidence:** evidence that the term is undefined; inference that it's material.

### EDGE-005 — Concurrent requests jointly exceed the balance *(likelihood high)*
**Description:** 5 days left. Employee submits 4 days, then another 4 before the
first is decided. Each passes validation *(ASSUMP-006)*.
**Why it matters:** Both approvals succeed and the balance goes to −3. Whether
negative balances are even representable is unspecified.
**Currently specified:** no.
**Confidence:** inference, dependent on ASSUMP-006 — if the balance is re-checked at
approval time this narrows to a race between two approvals rather than an ordinary
sequence of events. That distinction is worth confirming before building anything.

### EDGE-016 — Notification email bounces *(likelihood high, impact high)*
**Description:** The submission email doesn't arrive — filtered, bounced, or ignored.
**Why it matters:** §6 makes email the sole channel. There is no in-app queue badge
in any requirement, no delivery tracking, and no fallback. A silent delivery failure
is indistinguishable from a manager who hasn't looked yet, and it degrades into
EDGE-012.
**Currently specified:** no. → Q-013
**Confidence:** evidence — §6 states email; nothing states anything else.

### EDGE-013 — Employee with no manager *(likelihood medium, impact high)*
**Description:** New joiner before manager assignment, contractor, or the CEO.
**Why it matters:** ASSUMP-003 assumes exactly one line manager per employee. When
that's false the request routes nowhere — and it fails at submission, when the
employee is trying to do something legitimate, with no obvious recourse.
**Currently specified:** no. Related to Q-015.

---

## Not tested, and not testable here

**EDGE-014 — Employee leaves with approved future leave.** Whether the leave is
cancelled, paid out, or ignored is a policy question, not a system question. It needs
People Ops and possibly a payroll answer before it can be specified — and §7 puts
payroll out of scope, so it may be genuinely unanswerable in this project. Flagged
rather than resolved.
