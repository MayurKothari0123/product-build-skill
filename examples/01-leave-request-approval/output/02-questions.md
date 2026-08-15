> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete — GATE 1 cleared

# Questions

**Round 1** stopped at GATE 1 with 4 critical questions and generated nothing past
step 3. The product owner answered them; **round 2** ran steps 4–9 on those answers.

**7 high questions remain open.** Per GATE 2 these block a `PROCEED` verdict — see
`10-readiness.md`.

Answer inline on the `**Answer:**` line and set `status`, or answer in
conversation and FlowBreaker will write them back.

---

## Answered — round 1 criticals

These are frozen. Answers are carried as `evidence` in every downstream artifact.

---

### Q-005 · `critical` · scope · `answered`
**Who actually approves a leave request — the line manager, HR, or both in sequence?**

- **Why it matters:** §2 says "the employee's line manager approves or rejects it."
  §5 says "HR reviews and approves all leave requests." These cannot both be the
  sole approver. This is not a detail — it determines the entire approval flow, the
  permission model, the notification model, and how long a decision takes. Every
  flow below assumes an answer.
- **Blocks:** REQ-004, REQ-005, REQ-006, FLOW-002, FLOW-003
- **Answer type:** single_choice
- **Options:**
  - Manager only — §5 is stale
  - HR only — §2 is stale
  - Two-stage: manager decides, then HR confirms for compliance
  - Manager decides; HR only audits after the fact (no blocking step)
- **Status:** `answered`
- **Answer:** Manager decides; HR audits after the fact. §5 is stale text from an
  earlier draft and will be removed. HR does not block a decision.
- **Consequence:** REQ-004 clarity moves `contradictory` → `clear`. FLOW-003 (HR
  review) is deleted. C-2 is resolved. REQ-014's "fast" now has one approver to
  measure, not two.

---

### Q-001 · `critical` · permission · `answered`
**Can a manager approve their own leave request?**

- **Why it matters:** The source names managers as approvers but never excludes
  self-approval. A manager submitting leave is an ordinary case, not an edge case,
  and if unhandled the manager approves themselves. That is a permission hole and
  an audit finding in any organisation with a compliance function.
- **Blocks:** REQ-004, FLOW-002, TEST-010
- **Answer type:** single_choice
- **Options:**
  - No — escalates to the skip-level manager
  - No — escalates to HR
  - Yes, permitted
  - Yes below a threshold (e.g. ≤2 days), escalates above it
- **Status:** `answered`
- **Answer:** No. A manager's own request escalates to their skip-level manager.
- **Consequence:** New requirement REQ-015. New flow branch in FLOW-002. Raises a
  follow-on question nobody had asked: what happens for an employee with no
  skip-level — the CEO? → Q-015 (`high`, open below).

---

### Q-002 · `critical` · flow · `answered`
**What happens to a pending request when its approving manager is deactivated or leaves?**

- **Why it matters:** Nothing in the source describes the approver lifecycle. As
  written, the request sits pending forever while the employee sees "pending review"
  and believes it is being considered. This is silent data loss with a human
  consequence — someone takes leave that was never approved, or doesn't take leave
  they were entitled to.
- **Blocks:** FLOW-002, STATE-009, EDGE-004
- **Answer type:** single_choice
- **Options:**
  - Auto-reassign to the skip-level manager on deactivation
  - Escalate to HR after N days pending
  - Surface in an HR exception queue for manual reassignment
  - Block deactivation until pending requests are cleared
- **Status:** `answered`
- **Answer:** Auto-reassign to the skip-level manager on deactivation, and surface
  the reassignment in an HR exception view.
- **Consequence:** STATE-009 gains defined behaviour. EDGE-004 moves from
  `unspecified` to `specified`. TEST-011 can now assert a concrete result rather
  than merely that some behaviour exists.

---

### Q-004 · `critical` · security · `answered`
**Who can see another employee's leave records, and does leave type appear on the team calendar?**

- **Why it matters:** §3 says approved leave appears on the team calendar. Sick
  leave is one of the three types (§4). If the calendar shows leave type, the
  system publishes health-adjacent information about employees to their colleagues.
  That is a privacy problem and, in several jurisdictions, a legal one. The source
  never states what the calendar displays.
- **Blocks:** REQ-002, REQ-007, FLOW-004
- **Answer type:** multiple_choice
- **Options:**
  - Calendar shows "unavailable" only, no type
  - Calendar shows type for annual/unpaid, hides sick as "unavailable"
  - Calendar shows all types
  - Managers see type; peers see availability only
- **Status:** `answered`
- **Answer:** Calendar shows "unavailable" only. Leave type is visible to the
  employee, their manager, and HR — never to peers.
- **Consequence:** REQ-007 clarity moves `ambiguous` → `clear`. Removes the privacy
  finding. Adds a permission rule that was in nobody's requirements: the calendar
  and the request record need different visibility, so the calendar cannot simply
  render the request object.

---

## Open

### Q-003 · `high` · problem
**Is the primary objective HR's record accuracy, or the speed/consistency of approval decisions?**

- **Why it matters:** §1 frames an HR data problem; §8 measures success partly by
  employees no longer chasing. If speed is a real goal, §5's blocking HR review is
  the bottleneck you are removing, not a control to preserve. Changes the answer to
  Q-005.
- **Blocks:** PROB-001, PROB-002, REQ-014
- **Answer type:** single_choice
- **Options:** HR record accuracy · Approval speed · Both, accuracy first · Both, speed first
- **Status:** `open`
- **Answer:**

---

### Q-006 · `high` · data
**What is a "working day", and can leave be requested for past dates?**

- **Why it matters:** Balance deduction depends on counting days. Whether weekends
  and public holidays count — and whose public holidays, for a distributed team —
  changes every balance calculation. Separately, sick leave is almost always
  recorded retroactively, so if backdating is blocked the sick-leave case breaks.
- **Blocks:** REQ-002, REQ-006, REQ-009, EDGE-001
- **Answer type:** free_text
- **Status:** `open`
- **Answer:**

---

### Q-007 · `high` · ux
**Does a manager see team coverage or overlapping absences when deciding?**

- **Why it matters:** The stated flow gives a manager a request in isolation. The
  actual decision — "can we spare this person then?" — needs the team's existing
  approved leave. Without it the manager leaves the tool to check the calendar,
  which is the informal process the project is trying to replace.
- **Blocks:** FLOW-002, REQ-004
- **Answer type:** boolean
- **Status:** `open`
- **Answer:**

---

### Q-015 · `high` · flow · *raised by the answer to Q-001*
**Who approves leave for an employee with no skip-level manager?**

- **Why it matters:** Q-001's answer routes a manager's own request upward. For the
  CEO, or anyone reporting directly to them, there is no upward. Without a defined
  fallback the request escalates to nobody and strands — the exact failure Q-002 was
  answered to prevent, reintroduced by the fix for Q-001.
- **Blocks:** REQ-015, FLOW-002
- **Answer type:** single_choice
- **Options:** Falls back to HR · Falls back to a named board member · Self-approval
  permitted at that level · Configurable per employee
- **Status:** `open`
- **Answer:**

---

## Deferred to the next batch

Held back to keep round 1 at seven (§3, batch cap). Not less important than the
items above — just not blocking at the time.

| ID | Question | Risk | Blocks |
|---|---|---|---|
| Q-008 | Are half-days supported? | high | REQ-001, REQ-006 |
| Q-009 | Can an employee cancel or amend a request after submitting? Before or after approval? | high | FLOW-001, FLOW-005 |
| Q-010 | Does unused annual leave carry over at reset, and what happens to requests spanning the reset date? | high | REQ-009, EDGE-006 |
| Q-011 | Is sick leave unlimited, or capped by a policy not stated here? | medium | REQ-002, §4 |
| Q-012 | What is the HR export's format, trigger, and who can run it? No flow is defined for REQ-011. | medium | REQ-011 |
| Q-013 | If email notification fails, is there any fallback or retry? | medium | REQ-005, REQ-012 |
| Q-014 | Can HR override or reverse an approved request? Is it audited? | high | REQ-011, audit |

---

## Queue summary

| Status | Count | Of which critical | high |
|---|---|---|---|
| answered | 4 | 4 | — |
| open | 8 | 0 | 7 |
| assumed | 0 | — | — |
| deferred | 3 | 0 | 0 |

**Resolution: 4 of 12 (33%).**

Answering the four criticals cleared GATE 1 and unblocked steps 4–9 — but note that
one answer *created* a new open question (Q-015). Resolving a critical often exposes
the next layer down; a falling open-question count is not by itself a sign of
progress.

