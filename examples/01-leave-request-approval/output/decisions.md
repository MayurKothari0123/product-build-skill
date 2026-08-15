> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete — GATE 1 cleared

# Decisions

Every question asked, the answer given, and every assumption still standing. This is
the record of *why the system works the way it does* — the only file that outlives the
conversation it came from.

**Round 1** stopped at GATE 1 with 4 critical questions and generated nothing past
step 3. The product owner answered them; **round 2** ran steps 4–9 on those answers.
7 high questions remain open, which is why the verdict is `CLARIFY`.

---

## Answered

Frozen. Answers are carried as `evidence` in every downstream artifact and are never
rewritten — a change of mind is a new entry, not an edit (§4).

### Q-005 · was `critical` · scope
**Who actually approves a leave request — the line manager, HR, or both in sequence?**

§2 said "the employee's line manager approves or rejects it." §5 said "HR reviews and
approves all leave requests." These could not both describe the sole approver, and the
answer determines the approval flow, the permission model, the notification model and
how long a decision takes.

**Answer:** Manager decides; HR audits after the fact. §5 is stale text from an earlier
draft and will be removed. HR does not block a decision.
**Answered by:** product owner, in conversation
**Consequence:** REQ-004 moves `contradictory` → `clear`. FLOW-003 (HR review) deleted,
ID retired and never reused. REQ-014's "fast" now has one approver to measure, not two.

### Q-001 · was `critical` · permission
**Can a manager approve their own leave request?**

The source named managers as approvers but never excluded self-approval. A manager
submitting leave is an ordinary case, not an edge case — unhandled, the manager
approves themselves, which is a permission hole and an audit finding anywhere with a
compliance function.

**Answer:** No. A manager's own request escalates to their skip-level manager.
**Answered by:** product owner, in conversation
**Consequence:** Created REQ-015 and a new branch in FLOW-002. Raised a follow-on
question nobody had asked: what happens for an employee with no skip-level — the CEO?
→ **Q-015**, open below.

### Q-002 · was `critical` · flow
**What happens to a pending request when its approving manager is deactivated?**

Nothing in the source described the approver lifecycle. As written the request sits
pending forever while the employee sees "pending review" and believes it is being
considered — silent data loss with a human consequence.

**Answer:** Auto-reassign to the skip-level manager on deactivation, and surface the
reassignment in an HR exception view.
**Answered by:** product owner, in conversation
**Consequence:** STATE-009 gains defined behaviour. EDGE-004 moves `unspecified` →
`specified`. TEST-011 can assert a concrete result rather than merely that some
behaviour exists.

### Q-004 · was `critical` · security
**Who can see another employee's leave records, and does leave type appear on the
team calendar?**

§3 said approved leave appears on the team calendar. Sick leave is one of the three
types (§4). If the calendar showed type, the system would publish health-adjacent
information about employees to their colleagues.

**Answer:** Calendar shows "unavailable" only. Leave type is visible to the employee,
their manager and HR — never to peers.
**Answered by:** product owner, in conversation
**Consequence:** REQ-007 moves `ambiguous` → `clear`. Adds a permission rule that was
in nobody's requirements: the calendar and the request record need different
visibility, so the calendar cannot simply render the request object.

---

## Open

| ID | Question | Risk | Blocks |
|---|---|---|---|
| Q-006 | What is a "working day", and can leave be requested for past dates? | high | 6 items |
| Q-009 | Can a request be cancelled or amended, and at what point? | high | FLOW-005 entirely |
| Q-015 | Who approves for an employee with no skip-level? | high | 4 items |
| Q-010 | Carry-over at reset, and requests spanning the reset date | high | REQ-009, EDGE-006 |
| Q-008 | Are half-days supported? | high | REQ-001, REQ-006 |
| Q-007 | Does the manager see team coverage when deciding? | high | FLOW-002, REQ-004 |
| Q-014 | Can HR reverse an approved request? Is it audited? | high | REQ-011, audit |
| Q-003 | "The approval process should be fast" — fast meaning what number? | high | REQ-014 |
| Q-012 | HR export: format, trigger, who can run it? | medium | REQ-011 |
| Q-011 | Is sick leave unlimited, or capped by a policy not stated here? | medium | REQ-002 |
| Q-013 | If email notification fails, is there any fallback or retry? | medium | REQ-005 |

**Q-006 is the highest-leverage and the cheapest** — six blocked items, and someone in
People Ops already knows the answer.

---

## Assumptions

Gaps filled to keep the analysis moving. **These are not facts (R3).** Every artifact
that depends on one carries the dependency, and they reappear in the report.

| ID | Assumption | Source | Supports | If wrong |
|---|---|---|---|---|
| ASSUMP-001 | Whole days only; no half-day support | domain_convention | REQ-001, REQ-006 | Data-model rework; half-days are near-universal → Q-008 |
| ASSUMP-002 | One approver per request, not a chain | inferred | FLOW-002, REQ-004 | Superseded by the answer to Q-005 — retained for history |
| ASSUMP-003 | Employees have exactly one line manager | domain_convention | TEST-020, EDGE-013 | Matrix reporting routes requests nowhere |
| ASSUMP-004 | Team calendar is visible to the whole team | inferred | REQ-007, FLOW-004 | Determined the severity of Q-004, now answered |
| ASSUMP-005 | Notifications are email-only, per §6 | evidence-adjacent | EDGE-016, RISK-005 | Changes the severity of delivery failure |
| ASSUMP-006 | Balance checked at submission, not approval | inferred | TEST-006, EDGE-005 | The overdraw case changes shape entirely |
| ASSUMP-007 | Fixed-date annual reset, org-wide | inferred | REQ-009, EDGE-006 | Anniversary-based reset is a different feature |
| ASSUMP-008 | Rejected requests are terminal — no resubmission | inferred | FLOW-005 | A resubmission flow is missing entirely |

`source`: `inferred` · `domain_convention` · `evidence-adjacent` · `user_directed`

**8 of 8 are unvalidated.** None has been confirmed by a person, tested against real
usage, or supported by a document. None needs research — all eight are settled by one
conversation with People Ops.

### Assumptions the source document itself makes

Not gaps FlowBreaker filled — beliefs the document rests on without acknowledging.
These are findings about the PRD, not about the system.

| Embedded assumption | Where | Why it's worth surfacing |
|---|---|---|
| Managers respond promptly to email | §6 | The whole notification model is email-push with no escalation. The current process fails partly *because* people don't respond promptly; nothing here changes that. |
| HR reconciliation is the bottleneck worth fixing | §1 | Plausible, but §8's success criteria include employee chasing — which this design doesn't address. |
| Every employee has a manager in the system | §2 | Contractors, new joiners and the CEO all break this silently. |
| Leave policy is uniform across the organisation | §4 | No mention of location, seniority or contract type varying entitlement. |

---

## Resolution

| Status | Count | critical | high |
|---|---|---|---|
| answered | 4 | 4 | — |
| open | 11 | 0 | 8 |
| assumed | 0 | — | — |

**4 of 15 resolved (27%).** Answering the four criticals cleared GATE 1 and unblocked
steps 4–9 — but one answer *created* a new open question (Q-015). Resolving a critical
often exposes the next layer down; a falling open-question count is not by itself a
sign of progress.
