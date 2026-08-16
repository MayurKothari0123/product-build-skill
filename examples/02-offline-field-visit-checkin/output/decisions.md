> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete — GATE 1 overridden ("carry on"), criticals still open

# Decisions

## The problem, as restated

**PROB-001** — Field technicians record visits on paper handed in weekly. Forms get
lost, handwriting is unreadable, and Ops cannot see where anyone is during the day. The
cost lands in three different places: lost billing, unreadable records, and no
day-of visibility.

**PROB-002** — implied by §8 and served by no requirement: *"Ops knows where
technicians are."* Every requirement in §3 captures location **once**, at check-in.
Knowing where someone is during the day is continuous tracking, which is a different
product with different consent implications → **Q-002**.

**Non-goals (§7):** route planning, scheduling. Nothing else is excluded — notably,
neither employee-location policy nor billing correctness is addressed.

## Open — criticals

### Q-001 · `critical` · flow
**How does Ops see "near real time" status when the technician is offline all day?**

§2 states the app must work offline because many sites have no signal. §3 requires Ops
to see visit status in near real time. §4 says data syncs when connectivity returns. If
a technician spends the day in basements and rural sites — the exact cases §2 names —
Ops sees nothing until evening. The document requires both and reconciles neither.

- **a)** "Near real time" means *when synced* — honest, but §8's "Ops knows where
  technicians are" then fails on precisely the days it matters
- **b)** Opportunistic sync — the app pushes a lightweight status ping whenever any
  connectivity appears, separate from full visit sync
- **c)** Accept the gap and set expectations in the Ops UI: "last synced 3h ago"
- **d)** Something else

**Blocks:** REQ-006, REQ-007, FLOW-004, TEST-001, TEST-005

### Q-002 · `critical` · privacy · *coupled to Q-001*
**Is location captured once at check-in, or tracked continuously?**

§5 says check-in captures GPS to confirm the technician was on site. §8 says success
means Ops knows where technicians are. The first is an attendance record; the second is
continuous employee tracking. They are different systems, and the second requires a
consent and retention policy that appears nowhere in this document. Not askable before
Q-001 — if "near real time" is redefined, this may dissolve.

**Blocks:** REQ-002, REQ-006, PROB-002, TEST-006

### Q-003 · `critical` · data
**What happens when GPS is unavailable at check-in?**

§5 captures GPS "to confirm the technician was on site". §2 names basements and
industrial buildings as the reason the app must work offline — and those are exactly
the places GPS does not resolve. As written, the confirmation mechanism fails in the
primary use case.

- **a)** Check-in proceeds, location recorded as unavailable, flagged for Ops
- **b)** Check-in blocked until a fix is obtained — *breaks the basement case*
- **c)** Last known location is used with a staleness marker
- **d)** Something else

**Blocks:** REQ-002, EDGE-002, TEST-002

### Q-004 · `critical` · data
**What wins when the same visit is edited in two places before syncing?**

§4 says the technician "should not have to think about it". A technician edits a visit
on their phone while offline; Ops edits the same record; the phone syncs. Silent
resolution means someone's work disappears without either party knowing — and this
record is the billing input.

**Blocks:** REQ-007, REQ-008, FLOW-005, TEST-003

## Open — high

| ID | Question | Risk | Blocks |
|---|---|---|---|
| Q-005 | Timestamps come from the device clock, which the technician controls. Is that acceptable for billing? | high | REQ-002, REQ-005, REQ-008 |
| Q-006 | A technician never checks out — forgot, phone died, left site. What closes the visit? | high | REQ-005, EDGE-004 |
| Q-007 | Can a visit be edited after check-out? Before or after it has been billed? | high | REQ-008, EDGE-007 |
| Q-008 | Assigned visits are fetched when? A technician starting the day already offline has no list. | high | REQ-001, EDGE-001 |
| Q-009 | Does the billing export include unsynced visits, exclude them, or refuse to run? | high | REQ-008, EDGE-005 |
| Q-010 | Device lost or stolen with unsynced visits — recoverable, and is customer data on it wiped? | high | REQ-007, EDGE-009 |
| Q-011 | 10 photos per visit at what resolution, uploaded over whose data plan? | medium | REQ-004, EDGE-006 |
| Q-012 | How long are GPS coordinates retained, and who can query them historically? | high | REQ-002, Q-002 |

**Q-001 is the highest-leverage** — five blocked items, and it decides whether Q-002
is a privacy programme or a non-issue.

## Assumptions

Recorded to let the run continue past GATE 1. **None is a fact (R3).**

| ID | Assumption | Source | Supports | If wrong |
|---|---|---|---|---|
| ASSUMP-001 | One technician per visit | domain_convention | FLOW-002, TEST-003 | Two-person jobs need a different check-in model |
| ASSUMP-002 | A visit belongs to exactly one site | inferred | REQ-002 | Multi-building sites break the GPS confirmation |
| ASSUMP-003 | Sync is last-write-wins | domain_convention | FLOW-005 | This is the silent-data-loss case in Q-004 |
| ASSUMP-004 | Photos upload with the visit, not separately | inferred | REQ-004, EDGE-006 | A 10-photo visit may never complete on poor signal |
| ASSUMP-005 | Ops can edit visit records | inferred | Q-004, REQ-008 | If not, Q-004 collapses to a much simpler problem |
| ASSUMP-006 | Billing runs after sync completes | inferred | REQ-008 | Billing on partial data is the expensive failure |

### Assumptions the document itself makes

| Embedded assumption | Where | Why it's worth surfacing |
|---|---|---|
| Offline is the exception | §4 | §2 says "many sites have no signal". The document treats offline as a recovery path when the evidence says it is the normal path. |
| Technicians will check out | §3 | No requirement handles the visit that never closes, which paper forms handled by being handed in regardless. |
| GPS confirms presence | §5 | GPS is a claim about a device, not a person, and it does not resolve indoors — which is where the work happens. |
| Faster billing follows from digital records | §8 | Only if the records are complete and trusted. Q-004 and Q-009 both threaten that. |

## Resolution

| Status | Count | critical | high |
|---|---|---|---|
| answered | 0 | — | — |
| open | 12 | 4 | 7 |
| assumed | 6 | — | — |

**0 of 12 resolved.** Carried past GATE 1 rather than through it — a legitimate choice,
and a visible one. The verdict is capped at `CLARIFY` regardless of the rest.
