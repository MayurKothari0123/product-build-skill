> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete — GATE 1 overridden ("carry on"), criticals still open

# Decisions

## The problem, as restated

**PROB-001** — IoT sensors report to per-customer S3 buckets, processed by a nightly
batch. Alerts arrive up to 24 hours late, so a cold-storage failure has already spoiled
the stock by the time anyone knows. The cost is spoiled inventory, not latency itself.

**PROB-002** — implied but never stated: the customer needs to know a unit is failing
*while the stock can still be saved*. That window is a property of the physical goods,
not of the pipeline, and the document never names it. "One minute" appears to be a
round number rather than a derived one → **Q-005**.

**Non-goals (§8):** device firmware, device provisioning UI. Nothing else is excluded.

## Open — criticals

GATE 1 fired on these four. The run continued on an explicit "carry on", so everything
downstream rests partly on guesswork and the verdict cannot be `PROCEED` (R4, GATE 2).

### Q-001 · `critical` · data
**Is the one-minute alert SLA measured from the event's timestamp, or from its arrival
at the ingestion endpoint?**

§2 and §9 both promise alerts within one minute. §5 says some devices buffer events
locally and send them in a batch later. A buffered event arriving three hours late
cannot produce a one-minute alert under the first reading, and trivially can under the
second. These are two different products with the same requirement text.

- **a)** From receipt — *SLA is always met; a spoiled fridge can still go unnoticed for
  hours, which is the original problem*
- **b)** From event timestamp — *honest, but the SLA is then unachievable for buffering
  devices and the number needs restating*
- **c)** Two SLAs: real-time path and a backfill path, measured separately — *most
  likely correct; nothing in the document describes two paths*

**Blocks:** REQ-004, REQ-005, REQ-008, FLOW-003, TEST-001, TEST-004

### Q-002 · `critical` · data · *coupled to Q-001*
**When a buffered device delivers a three-hour-old batch, are alert rules evaluated
retroactively?**

"Temperature above 8°C for more than 10 minutes" (§6) was true three hours ago. Firing
now produces an alert about stock that is already spoiled; suppressing it means the
system silently knows about a failure it never reported. Not asked until Q-001 is
answered — its answer reframes this one entirely.

**Blocks:** REQ-004, FLOW-003, EDGE-002, TEST-004

### Q-003 · `critical` · security
**What binds an API key to a device ID and a customer?**

§7 says devices authenticate with an API key issued at provisioning. Nothing says the
key is scoped to one device or one customer. As written, any valid key can POST events
claiming any device ID — including another customer's. Today's per-customer S3 buckets
enforce that boundary structurally; a shared HTTP endpoint does not.

- **a)** Key is bound to one device ID; mismatched device ID is rejected
- **b)** Key is per-customer; device IDs are validated against that customer's fleet
- **c)** Key is a bearer token with no binding — *the current reading, and a
  cross-tenant data injection hole*

**Blocks:** REQ-001, REQ-002, FLOW-001, TEST-002

### Q-004 · `critical` · data
**Is a schema-invalid event "lost"?**

§9 states "no events lost" as a success criterion. §3 says events are validated against
a schema. The document never says what happens to an event that fails validation —
dropped, dead-lettered, or stored raw for inspection. Until that is decided, "no events
lost" cannot be measured, so it cannot be a success criterion.

**Blocks:** REQ-002, REQ-008, EDGE-004, TEST-003

## Open — high

| ID | Question | Risk | Blocks |
|---|---|---|---|
| Q-005 | "Within one minute" — derived from what? What is the actual spoilage window? | high | REQ-008, PROB-002 |
| Q-006 | What is "our expected volume"? Devices × 5-minute interval × customers = a number nobody wrote down | high | REQ-008, TEST-006 |
| Q-007 | Buffered devices resend on retry — are duplicate events deduplicated, and on what key? | high | REQ-002, EDGE-003 |
| Q-008 | What does "replay" mean — re-ingest, re-evaluate, or both? Does replay re-fire customer alerts? | high | REQ-007, FLOW-005 |
| Q-009 | Device clock skew: an event timestamped in the future, or before its device was provisioned | high | REQ-002, EDGE-005 |
| Q-010 | Alert delivery failure — email bounces, SMS undelivered. Retry, fallback, or silent drop? | high | REQ-005, EDGE-006 |
| Q-011 | Who may configure alert rules? §6 says "per customer", not which customer user | high | REQ-004 |
| Q-012 | What happens at the 90-day boundary — deleted, archived, or still queryable? | medium | REQ-006 |

**Q-001 is the highest-leverage.** Six blocked items, and it is the question the whole
alerting model rests on. It is also cheap: someone on Platform already has an opinion.

## Assumptions

Recorded to let the run continue past GATE 1. **None is a fact (R3).**

| ID | Assumption | Source | Supports | If wrong |
|---|---|---|---|---|
| ASSUMP-001 | SLA is measured from receipt, not event timestamp | inferred | FLOW-003, TEST-001 | The entire alerting model changes; Q-001 |
| ASSUMP-002 | One API key per device, issued at provisioning | domain_convention | FLOW-001, TEST-002 | Cross-tenant event injection is possible today |
| ASSUMP-003 | Events are immutable once stored | domain_convention | REQ-007, FLOW-005 | Replay semantics change entirely |
| ASSUMP-004 | Alert rules evaluate a single device's stream, not across devices | inferred | REQ-004, FLOW-003 | "Any two freezers in a site" rules need a different engine |
| ASSUMP-005 | Email and SMS are both sent, not one as fallback for the other | inferred | REQ-005, EDGE-006 | Changes the severity of a delivery failure |
| ASSUMP-006 | Device IDs are globally unique, not per-customer | domain_convention | REQ-001, Q-003 | Collision between two customers' device IDs |

`source`: `inferred` · `domain_convention` · `user_directed`

### Assumptions the document itself makes

| Embedded assumption | Where | Why it's worth surfacing |
|---|---|---|
| Devices can reach the network most of the time | §5 | §5 admits they cannot, then §2 promises a latency SLA that only holds when they can. The document contradicts itself across three sections without noticing. |
| One minute is fast enough | §2, §9 | Never derived. If a freezer spoils stock in 20 minutes, one minute and five minutes are equally fine — and the engineering cost between them is large. |
| Customers complaining is a proxy for the problem being solved | §9 | Complaints stop when alerting improves *or* when customers give up. |
| The nightly batch is the bottleneck | §1 | Plausible, but no measurement is given for where the 24 hours actually goes. |

## Resolution

| Status | Count | critical | high |
|---|---|---|---|
| answered | 0 | — | — |
| open | 12 | 4 | 7 |
| assumed | 6 | — | — |

**0 of 12 resolved.** This run was carried past GATE 1 rather than through it, which is
a legitimate choice and a visible one: every artifact here is provisional, and the
verdict is capped at `CLARIFY` regardless of what else is good.
