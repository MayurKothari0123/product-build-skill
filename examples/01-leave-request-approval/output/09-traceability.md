> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Traceability

## Requirements → Flows

| REQ | Statement | Flows | Gap |
|---|---|---|---|
| REQ-001 | Employee submits a request | FLOW-001 | — |
| REQ-002 | Balance shown to employee | FLOW-001 | — |
| REQ-003 | Manager sees team's pending requests | FLOW-002 | — |
| REQ-004 | Manager approves or rejects | FLOW-002 | — |
| REQ-005 | Employee notified of decision | FLOW-002 | — |
| REQ-006 | Balance deducted on approval | FLOW-002 | — |
| REQ-007 | Approved leave on team calendar | FLOW-004 | — |
| REQ-008 | Manager notified on submission | FLOW-001 | — |
| REQ-009 | Balances reset annually | **none** | No flow. No actor. No trigger. Orphaned. |
| REQ-010 | Rejection requires a reason | FLOW-002 | — |
| REQ-011 | HR exports monthly report | **none** | No flow defined → Q-012 |
| REQ-012 | Sick leave requires no balance | FLOW-001 | — |
| REQ-013 | Unpaid leave doesn't affect balance | FLOW-001 | — |
| REQ-014 | "Approval should be fast" | — | Not a requirement; replaced in 07 |
| REQ-015 | Manager's own request escalates | FLOW-002 | Added by Q-001's answer |

**Coverage: 13 of 15 requirements map to a flow (87%).**

The two that don't are the two nobody has thought through. REQ-009 is the more
interesting: an annual balance reset has no user, no screen, and no trigger anyone
has described, which is why it slipped past — it isn't a feature, it's a scheduled
job that quietly rewrites everyone's entitlement.

## Flows → Requirements

| Flow | Requirements | Orphan? |
|---|---|---|
| FLOW-001 Submit | REQ-001, REQ-002, REQ-008, REQ-012, REQ-013 | no |
| FLOW-002 Decide | REQ-003, REQ-004, REQ-005, REQ-006, REQ-010, REQ-015 | no |
| FLOW-004 Calendar | REQ-007 | no |
| FLOW-005 Cancel/amend | **none** | **Yes — flow with no requirement.** Not in the source at all; included because users will attempt it. → Q-009 |

## Requirements → Tests

| REQ | Tests | Coverage |
|---|---|---|
| REQ-001 | TEST-009 | boundary only — no happy path written |
| REQ-002 | TEST-006 | boundary only |
| REQ-003 | TEST-020 | error only |
| REQ-004 | TEST-010, TEST-011, TEST-012, TEST-018 | permission, concurrency, recovery |
| REQ-005 | TEST-021 | error only |
| REQ-006 | TEST-012, TEST-013 | concurrency, boundary |
| REQ-007 | TEST-007 | security |
| REQ-008 | TEST-021 | error only |
| REQ-009 | **none** | uncovered |
| REQ-010 | **none** | uncovered — TEST-004 identified, not written |
| REQ-011 | **none** | uncovered |
| REQ-012 | TEST-009 | boundary |
| REQ-013 | **none** | uncovered |
| REQ-014 | **none** | n/a — replaced |
| REQ-015 | TEST-010, TEST-011 | permission |

**Coverage: 9 of 15 (60%).**

Note the shape of this coverage: it is heavily weighted to error, permission and
concurrency, and thin on happy paths — because happy-path tests were deliberately
deprioritized (§3 step 8). That is intentional, not an oversight, but it means this
suite is **not** a substitute for a normal test suite. It is the set of tests nobody
would have written.

## Questions → Blocked items

| Q | Risk | Status | Blocks |
|---|---|---|---|
| Q-005 | critical | answered | REQ-004, REQ-005, REQ-006, FLOW-002 — **unblocked** |
| Q-001 | critical | answered | REQ-004, FLOW-002, TEST-010 — **unblocked** |
| Q-002 | critical | answered | FLOW-002, STATE-009, EDGE-004 — **unblocked** |
| Q-004 | critical | answered | REQ-002, REQ-007, FLOW-004 — **unblocked** |
| Q-003 | high | open | PROB-001, PROB-002, REQ-014 |
| Q-006 | high | open | REQ-002, REQ-006, REQ-009, EDGE-001, TEST-009, TEST-013 |
| Q-007 | high | open | FLOW-002, REQ-004 |
| Q-008 | high | open | REQ-001, REQ-006 |
| Q-009 | high | open | FLOW-005 (entirely), STATE-013, STATE-020–023 |
| Q-010 | high | open | REQ-009, EDGE-006, TEST-014 |
| Q-014 | high | open | REQ-011, audit trail |
| Q-015 | high | open | REQ-015, FLOW-002, STATE-011, TEST-020 |
| Q-012 | medium | open | REQ-011 |
| Q-013 | medium | open | REQ-005, REQ-012, EDGE-016, TEST-021 |

**Q-006 is the highest-leverage open question** — it blocks 6 items across
requirements, edge cases and tests. It is also the cheapest to answer: someone in HR
knows what a working day is.

## Assumptions → Dependants

| ASSUMP | Status | Depended on by |
|---|---|---|
| ASSUMP-001 half-days unsupported | unvalidated | REQ-001, REQ-006, FLOW-001 |
| ASSUMP-002 single approver | unvalidated | FLOW-002, REQ-004 — **partly resolved** by Q-005 |
| ASSUMP-003 one line manager | unvalidated | FLOW-002, REQ-003, EDGE-013, TEST-020 |
| ASSUMP-004 calendar team-wide | unvalidated | REQ-007, FLOW-004 — **resolved** by Q-004 |
| ASSUMP-005 email-only notification | unvalidated | REQ-005, REQ-012, EDGE-016 |
| ASSUMP-006 balance checked at submission | unvalidated | REQ-002, REQ-006, EDGE-005, TEST-006 |
| ASSUMP-007 fixed-date annual reset | unvalidated | REQ-009, EDGE-006 |
| ASSUMP-008 rejection is terminal | unvalidated | FLOW-002, FLOW-005 |

**6 of 8 assumptions remain unvalidated and are load-bearing.** ASSUMP-006 and
ASSUMP-003 each support a test that would otherwise have no basis.

## Reference integrity

Checked automatically as the final step of the run (§3 step 9).

| Check | Result |
|---|---|
| Dangling references | **none** — every referenced ID exists |
| Requirements with no flow | 2 — REQ-009, REQ-011 *(reported, not patched)* |
| Flows missing a required state | 1 — FLOW-004 has no error state and no N/A |
| Flows with no requirement | 1 — FLOW-005 *(expected; it isn't in the source)* |
| Critical findings with no test | none |
| Retired IDs | FLOW-003 — deleted after Q-005; **ID not reused** |
| Questions referencing non-existent items | none |

FLOW-004's missing error state was found by this check, not by the flow analysis
that produced FLOW-004. That is the integrity check doing its job: the analysis
step is where oversights happen, so the check exists to catch its author.
