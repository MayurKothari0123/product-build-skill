> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Acceptance Criteria Review

**The source document contains no acceptance criteria.** §3 lists requirements as
capability statements ("Employees can submit a leave request") and §8 lists outcomes,
three of the four unmeasurable.

So this section reviews what stands in for acceptance criteria and proposes real
ones for the highest-value requirements. Nothing here is a finding *against* the
authors — a v0.3 draft is not expected to have them. It is a statement of what must
exist before anyone can say the feature works.

| REQ | Stated as | Testable | Unambiguous | States result | Covers failure | Verdict |
|---|---|---|---|---|---|---|
| REQ-001 | "Employees can submit a leave request..." | partly | no | no | no | rewrite |
| REQ-002 | "The system shows the employee their remaining balance" | partly | no | no | no | rewrite |
| REQ-004 | "Managers can approve or reject a request" | partly | yes *(post Q-005)* | no | no | rewrite |
| REQ-006 | "Approved leave is deducted from the balance" | yes | no | yes | no | rewrite |
| REQ-007 | "Approved leave appears on the team calendar" | yes | yes *(post Q-004)* | yes | no | augment |
| REQ-009 | "Leave balances reset annually" | no | no | no | no | **cannot write — blocked on Q-010** |
| REQ-011 | "HR can export a monthly report" | no | no | no | no | **cannot write — blocked on Q-012** |
| REQ-014 | "The approval process should be fast" | no | no | no | no | replace |

---

## Rewrites

### REQ-001 — Submit a leave request
**Was:** Employees can submit a leave request specifying start date, end date and leave type.

**Issues:** No preconditions. No expected result. No validation rules. No failure
behaviour. "Can submit" is satisfied by a button that does nothing.

**Suggested:**
- Given an employee with sufficient balance, when they submit a request for future
  dates, then a request is created with status `pending`, the employee sees a
  confirmation stating who will decide, and the manager is notified within 60s.
- Given an employee with insufficient annual-leave balance, when they submit, then
  submission is blocked with a message stating days requested and days available.
- Given a request for sick leave, when they submit, then no balance check is applied
  *(REQ-012)*.
- Given a request whose dates overlap an existing pending or approved request, when
  they submit, then **— unspecified, EDGE-010.**
- Given a submission that fails on the server, when the error returns, then the form
  retains all entered values and offers retry *(currently unspecified, STATE-006)*.

### REQ-004 — Approve or reject
**Was:** Managers can approve or reject a request. Rejection requires a reason.

**Issues:** Silent on self-approval (resolved by Q-001), on double submission, and on
what the manager sees when the request was already decided.

**Suggested:**
- Given a pending request from a direct report, when the manager approves, then
  status becomes `approved`, the balance is deducted *(REQ-006)*, and the employee is
  notified within 60s.
- Given a rejection with an empty reason, when submitted, then it is blocked
  *(REQ-010)*.
- Given a request already decided by another approver, when this manager submits a
  decision, then it is rejected with the existing decision and decider shown
  *(STATE-012)*.
- Given a manager submitting their own request, when created, then it is routed to
  their skip-level manager and does not appear in their own queue *(Q-001)*.
- Given the same approval submitted twice, when the second arrives, then it is a
  no-op and the balance is deducted once *(EDGE-007 — requires idempotency, which no
  requirement currently states)*.

### REQ-006 — Balance deduction
**Was:** Approved leave is deducted from the balance.

**Issues:** Doesn't say how many days a request represents. That is EDGE-002 and
Q-006, and it is the whole difficulty.

**Suggested:**
- Given an approved annual-leave request, when approval completes, then the balance
  decreases by the number of **working days** in the range — where "working day"
  excludes weekends and public holidays **for that employee's location** *(blocked on
  Q-006)*.
- Given approved sick or unpaid leave, when approval completes, then the annual
  balance is unchanged *(REQ-012, REQ-013)*.
- Given two approvals that would take the balance below zero, when the second
  completes, then **— unspecified, EDGE-005.**

### REQ-014 — Replace entirely
**Was:** "The approval process should be fast."

Not a requirement. Suggested replacements, of which the second is the one that
matters:
- 90% of requests receive a decision within 2 working days of submission.
- The employee can see the current status and elapsed waiting time at any point
  without contacting anyone. *(This addresses PROB-002, which no current requirement
  addresses at all.)*

---

## §8 success criteria

| Stated | Verdict | Suggested |
|---|---|---|
| "HR no longer reconciles leave manually" | **Keep.** The only measurable one. | Add the baseline: ~2 days/month today, target 0. |
| "Employees stop chasing managers" | Real outcome, no measure | Median time-to-decision < 2 working days, and status visible without asking. |
| "Intuitive and easy to use" | Not a criterion | A new employee completes a first request unaided in under 90s, n≥5 usability sessions. |

---

## Coverage

- Requirements with acceptance criteria in the source: **0 of 14**
- Requirements with criteria proposed here: **6**
- Requirements where criteria **cannot** be written yet: **2** (REQ-009, REQ-011) —
  both blocked on open questions, not on effort
- Requirements not yet addressed: **6** — lower priority, available on request

The two blocked ones are worth noting: you cannot write acceptance criteria for the
annual reset or the HR export because nobody has decided what they do. That is a
specification gap wearing the costume of a documentation gap.
