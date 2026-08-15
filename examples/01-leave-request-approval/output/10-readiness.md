> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Readiness Report

## Verdict: `CLARIFY`

**7 high-risk questions remain open, blocking 15 downstream items.** All four
critical questions from round 1 are answered, which is real progress — the approval
model is now coherent. But six of eleven test cases still cannot state an expected
result, and a specification you cannot write a passing test against is not ready to
implement.

Per GATE 2: no critical questions are open, so `PROCEED` is not blocked on that
count — it is blocked by the 3+ open `high` questions rule.

---

## Blocking questions

| Q | Question | Blocks |
|---|---|---|
| Q-006 | What is a "working day", and can leave be backdated? | 6 items |
| Q-009 | Can a request be cancelled or amended, and when? | FLOW-005 entirely |
| Q-015 | Who approves for an employee with no skip-level? | 4 items |
| Q-010 | Carry-over and requests spanning the reset date | REQ-009, EDGE-006 |
| Q-008 | Are half-days supported? | REQ-001, REQ-006 |
| Q-007 | Does the manager see team coverage when deciding? | FLOW-002 |
| Q-014 | Can HR reverse an approval? Audited? | REQ-011, audit |

Q-006 is the highest-leverage and the cheapest — someone in HR already knows the
answer.

---

## Findings by severity

```
Critical  0   ← all resolved in round 1
High     12   ← fix before implementation
Medium   14   ← fix before release
Low       4   ← track
```

### High-risk findings

| ID | Finding | Type | Basis |
|---|---|---|---|
| RISK-001 | No escalation, reminder or SLA on a pending request — the original problem, rebuilt | product | inference |
| RISK-002 | Approval is not idempotent; browser-back double-approves and double-deducts | technical | inference |
| RISK-003 | "Working day" undefined; every balance calculation is unspecified | product | evidence |
| RISK-004 | No cancel/amend path; users will email HR, recreating the manual process | ux | inference |
| RISK-005 | Email is the sole notification channel with no delivery detection or fallback | technical | evidence |
| RISK-006 | Concurrent pending requests can jointly overdraw the balance | technical | inference |
| RISK-007 | No audit trail, though §5 invokes "policy compliance" | compliance | evidence |
| RISK-008 | REQ-009 (annual reset) has no flow, actor or trigger | product | evidence |
| RISK-009 | REQ-011 (HR export) has no flow — and it is the requirement that serves the stated primary problem | product | evidence |
| RISK-010 | No retention or deletion policy for leave records including sick leave | privacy | evidence |
| RISK-011 | Employees with no manager (new joiners, contractors, CEO) route nowhere | technical | inference |
| RISK-012 | 14 of 14 requirements have no acceptance criteria | product | evidence |

**RISK-009 deserves emphasis.** §1 says the problem is HR's manual reconciliation.
REQ-011 — the HR export — is the requirement that solves it. It has no flow, no
format, no trigger, no permissions, and no test. The document's central problem is
served by its least-specified requirement.

---

## Coverage

| Metric | Value | |
|---|---|---|
| Requirements with a flow | 13 / 15 | 87% |
| Flows with empty + loading + error states | 3 / 4 | 75% |
| Requirements with a test | 9 / 15 | 60% |
| Questions resolved | 4 / 12 | 33% |
| Assumptions validated | 2 / 8 | 25% |
| Tests that can state an expected result | 5 / 11 | 45% |

**Coverage measures whether we looked, not whether the thing is good.** 87%
requirement-to-flow coverage on requirements that are 7/15 ambiguous is 87% coverage
of ambiguity.

The last row is the one to watch. When most of your tests can only assert *that some
behaviour exists*, the specification is not close to done regardless of the other
numbers.

---

## Untested assumptions

Six unvalidated assumptions are load-bearing:

| ASSUMP | Supports | If wrong |
|---|---|---|
| ASSUMP-006 balance checked at submission | TEST-006, EDGE-005 | The overdraw case changes shape entirely |
| ASSUMP-003 one line manager per employee | TEST-020, EDGE-013 | Requests route nowhere for matrix-reporting staff |
| ASSUMP-001 no half-days | REQ-001, REQ-006 | Data model rework; half-days are near-universal |
| ASSUMP-005 email-only notifications | EDGE-016, RISK-005 | Changes the severity of delivery failure |
| ASSUMP-007 fixed-date annual reset | REQ-009, EDGE-006 | Anniversary-based reset is a different feature |
| ASSUMP-008 rejection is terminal | FLOW-005 | A resubmission flow is missing entirely |

None of these needs research. All six are settled by one conversation with People Ops.

---

## Product readiness

**Not ready.** The problem is well stated and genuinely worth solving — §1 is
specific, quantified, and grounded in observed pain, which is more than most drafts
manage.

Three product-level gaps:

1. **The primary problem's requirement is the least specified.** RISK-009, above.
2. **The employee's problem is measured but not addressed.** §8 wants employees to
   stop chasing. No requirement gives an employee visibility into status or elapsed
   time, and none creates escalation. PROB-002 is real and currently unserved.
3. **One non-goal.** Only payroll is excluded. Half-days, carry-over, delegation,
   multi-approver chains and public-holiday handling are all undiscussed, and a
   reader could reasonably assume any of them is in scope.

## UX readiness

**Not ready.** States were largely absent from the source and have been generated
here rather than specified.

- **Empty states:** covered for 3 of 4 flows.
- **Loading:** covered, with one significant gap — FLOW-004's loading and empty
  calendars are indistinguishable, and the wrong reading causes a booking clash.
- **Error:** 4 of 9 error states have no defined behaviour.
- **Recovery:** weakest area. No path back from a failed submit, a failed decision
  save, or a request nobody acts on.
- **Accessibility:** not assessed — there is no interface yet. TEST-024 and TEST-025
  identified for when there is.

---

## What we still don't know

Known unknowns. Not findings, not risks — open uncertainty.

- Whether the real objective is HR accuracy or approval speed (Q-003). The two imply
  different products and the document supports both readings.
- Whether managers will actually decide faster in a tool than over WhatsApp. Nothing
  in this design changes the incentive; it changes the interface.
- Whether employees consider the current process a problem at all, or only HR does.
  §8 asserts they do; no evidence is cited.
- Whether leave policy is uniform across locations. Never mentioned; rarely true.
- What the HR export actually needs to contain to replace the spreadsheet. Nobody has
  described the spreadsheet.

---

## Recommended next actions

1. **Answer Q-006 (working day, backdating) — People Ops, 10 minutes.** Unblocks 6
   items and 2 tests. Highest leverage per unit effort in this list.
2. **Specify REQ-011, the HR export — People Ops + whoever owns the spec.** It is the
   requirement that solves the stated problem and it is currently one sentence. Start
   by looking at the spreadsheet it replaces.
3. **Decide the escalation model — product owner.** RISK-001. Without it the system
   reproduces the problem it was built to fix.
4. **Answer Q-009 (cancel/amend) — product owner.** An entire flow is missing.
5. **Validate ASSUMP-001, 003, 005, 006, 007 — one 30-minute conversation with
   People Ops.** Five load-bearing assumptions, one meeting.
6. **Add acceptance criteria to REQ-001, 004, 006 — spec owner.** Drafts in
   `07-acceptance-review.md`.
7. **Then re-run FlowBreaker.** Expect the verdict to move to `PROCEED` or
   `USER-TEST` depending on whether Q-003 resolves toward speed.

---

## How to read this report

| Label | Means |
|---|---|
| **Evidence** | Quoted or directly cited from `prd.md` |
| **Inference** | Reasoned from the document; reasoning shown |
| **Assumption** | A gap filled without support — treat as unverified |
| **Recommendation** | A judgement about what should happen, not a finding |
| **Unresolved** | Nobody has answered this yet |

This is an AI-generated analysis of a document. It finds what the document fails to
say. **It cannot tell you what your users actually need** — every question about
whether employees find the current process painful, whether managers will respond
faster, and whether the HR export matches the real spreadsheet requires talking to
those people.
