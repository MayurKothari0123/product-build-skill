> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Acceptance Criteria

**The source contains none.** §9 offers three success criteria; two of them cannot be
tested and the third cannot be measured without a decision nobody has made.

| REQ | Requirement | Testable | Unambiguous | States result | Covers failure | Verdict |
|---|---|---|---|---|---|---|
| REQ-001 | Devices send events to an ingestion endpoint | partly | no | no | no | rewrite |
| REQ-002 | Events validated against a schema | partly | no | no | **no** | rewrite |
| REQ-003 | Valid events stored and available in the dashboard | no | no | no | no | rewrite |
| REQ-004 | Events evaluated against configurable alert rules | no | no | no | no | rewrite |
| REQ-005 | Alerts delivered by email and SMS | partly | no | no | **no** | rewrite |
| REQ-006 | Customers view event history for the last 90 days | yes | partly | partly | no | tighten |
| REQ-007 | Ops can replay events for a device over a time range | no | **no** | no | no | rewrite |
| REQ-008 | "The pipeline should handle our expected volume" | **no** | **no** | **no** | no | **unfalsifiable** |

## §9's success criteria, assessed

| Stated | Verdict | Why |
|---|---|---|
| "Alerts arrive within one minute" | **unmeasurable as written** | One minute from *what* is undecided (Q-001), and "arrive" is unverified — nothing tracks delivery (EDGE-006). |
| "No events lost" | **unfalsifiable** | "Lost" is undefined. A schema-rejected event may or may not count (Q-004). Without that, the criterion can never fail. |
| "Customers stop complaining about late alerts" | **not a criterion** | Complaints also stop when customers give up. Measures sentiment, attributes it to one cause. |

---

## Rewrites

### REQ-008 — the volume requirement
**Was:** "The pipeline should handle our expected volume."
**Issues:** No number, no unit, no time window, no failure behaviour. Cannot pass or
fail, so it cannot be built against or tested.
**Blocked by:** Q-006 — nobody has stated the expected volume.
**Suggested shape**, once Q-006 is answered:
- Given N devices reporting every 5 minutes, when sustained for 24 hours, then p99
  ingestion latency stays under X ms and no event is rejected for capacity.
- Given a 10× burst when all devices at a site reconnect after an outage (EDGE-015),
  when the burst lasts 5 minutes, then events are accepted or explicitly backpressured
  — never silently dropped.

### REQ-002 — validation
**Was:** "Events are validated against a schema."
**Issues:** §4 says payload shape depends on event type, so "a schema" is really a
schema per type — none defined. No behaviour on failure.
**Suggested:**
- Given an event whose payload matches its declared type, when POSTed with a valid key,
  then it is accepted and acknowledged with the stored event ID.
- Given an event whose payload does not match its declared type, when POSTed, then it
  is rejected with a machine-readable error naming the failing field, **and retained
  for inspection** — unless "no events lost" excludes invalid events, which is
  undecided → **Q-004**.

### REQ-005 — alert delivery
**Was:** "Alerts are delivered by email and SMS."
**Issues:** No delivery confirmation, no retry, no fallback, no dedup, no cooldown. As
written, "delivered" means "sent".
**Suggested:**
- Given an alert rule that transitions to firing, when it fires, then exactly one
  notification is dispatched per channel and no further notification for the same rule
  and device until it resolves or a cooldown of X minutes elapses (EDGE-017).
- Given a dispatch that fails, when the provider reports failure or times out, then it
  is retried with backoff and the failure is visible in the alert's delivery log
  (EDGE-006).

### REQ-007 — replay
**Was:** "Ops can replay events for a device over a time range."
**Issues:** "Replay" is undefined — re-ingest, re-evaluate, or both — and the most
natural implementation pages customers (EDGE-010).
**Blocked by:** Q-008. Cannot be written until replay's purpose is decided.

### REQ-006 — history retention
**Was:** "Customers can view event history for the last 90 days."
**Issues:** Closest to testable in the document. Unstated: what happens at the boundary
— deleted, archived, or queryable-but-slow — and whether 90 days counts from event
timestamp or ingestion.
**Suggested:**
- Given events spanning 120 days, when a customer queries the full range, then events
  from the last 90 days are returned and the response states explicitly that earlier
  events exist but are outside the retention window — rather than returning a silently
  truncated list (EDGE-014).

---

## Requirements with no acceptance criteria

**All eight.** The document states what the system *is* and almost never what it should
*do* when something is absent, late, duplicated, malformed or failing.
