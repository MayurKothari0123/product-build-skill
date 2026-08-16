> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Edge Case Register

18 identified. **0 are specified in the source.** §5 mentions buffering devices and §7
mentions API keys; neither section says what the system does about them.

| ID | Edge case | Category | Likelihood | Impact | Specified? | Test |
|---|---|---|---|---|---|---|
| EDGE-001 | Buffered device delivers a 3-hour batch after reconnecting | timing | high | critical | no | TEST-004 |
| EDGE-002 | Alert condition was true in the past, discovered on backfill | timing | high | critical | no | TEST-004 |
| EDGE-003 | Device retries a batch already accepted — duplicate events | data | high | high | no | TEST-005 |
| EDGE-004 | Event fails schema validation | data | high | high | no | TEST-003 |
| EDGE-005 | Event timestamp is in the future (clock skew) | data | medium | high | no | TEST-007 |
| EDGE-006 | Alert email bounces or SMS is undelivered | network | high | critical | no | TEST-008 |
| EDGE-007 | Valid API key posts events for another customer's device ID | permission | medium | critical | no | TEST-002 |
| EDGE-008 | Device battery dies mid-batch; partial POST | network | medium | medium | no | — |
| EDGE-009 | Alert rule edited while an alert condition is mid-evaluation | timing | medium | high | no | — |
| EDGE-010 | Ops replay re-fires customer-facing alerts | permission | medium | critical | no | TEST-009 |
| EDGE-011 | Decommissioned device keeps sending events | data | medium | medium | no | — |
| EDGE-012 | Two customers provisioned with the same device ID | data | low | critical | no | — |
| EDGE-013 | Payload shape doesn't match its declared event type | data | high | medium | no | TEST-003 |
| EDGE-014 | Query spans the 90-day retention boundary | boundary | high | medium | no | TEST-010 |
| EDGE-015 | All devices at a site reconnect simultaneously after an outage | scale | medium | high | no | TEST-006 |
| EDGE-016 | Dashboard loads while backfill is in progress — partial history | timing | high | medium | no | — |
| EDGE-017 | Alert fires repeatedly while a condition persists (no dedup/cooldown) | timing | high | high | no | — |
| EDGE-018 | API key leaked from a physically accessible sensor | permission | medium | critical | no | — |

`category`: `data|timing|permission|network|input|scale`

---

## Detail — EDGE-001 · buffered batch delivery

**Description:** A sensor loses connectivity, buffers events locally for three hours per
§5, reconnects, and POSTs the batch.

**Why it matters:** This is the case the entire design has to answer and the one it
never mentions. Every promise in §2 and §9 — one-minute alerts, no events lost —
either holds or collapses depending on what happens here, and the document gives no
way to tell which. It is not an exotic case: §5 states it as normal behaviour for some
models.

**Currently specified:** no
**Confidence:** evidence — §5 states the buffering; the gap is that nothing else
acknowledges it.
**Suggested handling:** Two paths with separate SLAs — real-time events alert within
one minute of receipt; backfilled events are stored, evaluated on a distinct path, and
alert with an explicit "detected late" marker. Product decision → **Q-001**, **Q-002**.

---

## Detail — EDGE-006 · alert delivery failure

**Description:** The alert is generated correctly and the email bounces, or the SMS
never arrives.

**Why it matters:** Every requirement in this document is about producing an alert. Not
one is about the alert *arriving*. The success criterion "alerts arrive within one
minute" (§9) cannot be measured at all without delivery confirmation, and the failure
is silent on both ends — the customer doesn't know an alert existed, and the platform
doesn't know it failed. The old nightly batch at least left a file behind.

**Currently specified:** no
**Confidence:** inference — §3 names email and SMS as channels and stops there.
**Suggested handling:** Delivery receipts, retry with backoff, escalation to the second
channel on failure, and a visible delivery log per alert. → **Q-010**

---

## Detail — EDGE-010 · replay re-fires customer alerts

**Description:** Ops replays a device's events over a time range (REQ-007) to
investigate an issue. The replayed events pass through rule evaluation and dispatch
real alerts to the customer.

**Why it matters:** An internal debugging action producing customer-visible pages at
3am is the kind of defect that gets found in production exactly once, memorably. REQ-007
says "Ops can replay events" and nothing more; the natural implementation — push the
events back through the pipeline — has this behaviour by default.

**Currently specified:** no
**Confidence:** inference — from REQ-007 having no stated boundary.
**Suggested handling:** Replay is a distinct mode with alerting disabled, or writes to
a shadow evaluation path. Needs a product decision on what replay is *for* → **Q-008**.

---

## Detail — EDGE-017 · alert storm

**Description:** A freezer fails at 02:00. The condition "above 8°C for more than 10
minutes" stays true until someone fixes it. Rules are evaluated on every event, and
events arrive every 5 minutes.

**Why it matters:** Without deduplication or a cooldown, one failure produces an alert
every five minutes for eight hours — roughly 96 SMS messages for a single incident. The
customer silences the channel, and the next real alert is missed. The document
specifies rule evaluation and delivery, and nothing about alert lifecycle: firing,
re-firing, acknowledging, resolving.

**Currently specified:** no
**Confidence:** inference — §6 describes a rule as a condition, with no state attached.
**Suggested handling:** Alerts need a lifecycle (`firing` → `acknowledged` → `resolved`)
with dedup on rule + device and a cooldown. This is a missing requirement, not an edge
case of an existing one.
