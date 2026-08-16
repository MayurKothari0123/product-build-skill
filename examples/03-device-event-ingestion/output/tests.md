> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Test Cases

10 written at `critical` and `high` priority. **7 of 10 are blocked** — they cannot
state an expected result because nobody has decided what should happen. That ratio is
the clearest single measure of this spec's readiness.

Happy paths are deliberately deprioritised. This is the set of tests nobody would
otherwise write, not a substitute for a normal suite.

---

### TEST-001 · `critical` · timing · **blocked**
**One-minute SLA for a real-time event**

- **Preconditions:** Device D provisioned and online. Alert rule R active for D.
- **Steps:**
  1. Emit an event from D that satisfies R.
  2. Record the event timestamp, the receipt timestamp and the alert dispatch time.
- **Expected results:** Alert dispatched within 60 seconds of — **undecided**.
- **Related:** REQ-004, REQ-005 · FLOW-003
- **Blocked by:** Q-001. Measured from event timestamp or receipt? The test asserts
  only that *some* bound is enforced, because writing 60s from either would invent the
  answer.

### TEST-002 · `critical` · security
**API key cannot post events for another customer's device**

- **Preconditions:** Customer A with device D-A and key K-A. Customer B with device D-B.
- **Steps:**
  1. POST an event with key `K-A` and device ID `D-B`.
  2. Query customer B's dashboard for that time range.
  3. Query customer A's dashboard for the same range.
- **Expected results:**
  1. The POST is rejected — 403, not 200.
  2. No event appears under customer B.
  3. No event appears under customer A either; it is not silently reattributed.
- **Related:** REQ-001 · FLOW-001 · EDGE-007 · **Q-003**
- **Note:** This test is expected to **fail** against a faithful implementation of the
  PRD. §7 issues an API key and never scopes it. The failure *is* the finding.

### TEST-003 · `high` · error · **blocked**
**Schema-invalid event**

- **Steps:** POST an event whose payload does not match its declared type.
- **Expected results:** Rejected with a machine-readable error naming the failing
  field. Whether it is retained is **undecided**.
- **Related:** REQ-002 · EDGE-004, EDGE-013
- **Blocked by:** Q-004 — is a rejected event "lost" for the purposes of §9?

### TEST-004 · `critical` · timing · **blocked**
**Buffered device delivers a three-hour batch**

- **Preconditions:** Device D buffers offline for 3 hours. One buffered event satisfies
  alert rule R.
- **Steps:**
  1. Restore connectivity; D POSTs the batch.
  2. Observe alert dispatch and its stated timestamp.
- **Expected results:** All events stored. Whether R fires retroactively, is suppressed,
  or fires with a "detected late" marker is **undecided**.
- **Related:** REQ-004 · FLOW-003 · EDGE-001, EDGE-002
- **Blocked by:** Q-001, Q-002. This is the single most important test in the suite and
  it cannot be written.

### TEST-005 · `high` · data · **blocked**
**Duplicate batch on retry**

- **Steps:** POST an identical batch twice, simulating a retry after a timeout.
- **Expected results:** Events appear once, not twice. Dedup key **undecided**.
- **Related:** REQ-002 · EDGE-003
- **Blocked by:** Q-007 — dedup on device+timestamp, on an event ID the device does not
  currently send, or not at all.

### TEST-006 · `high` · boundary · **blocked**
**Simultaneous reconnect burst**

- **Steps:** All devices at one site reconnect at once and flush buffers together.
- **Expected results:** Accepted or explicitly backpressured — never silently dropped.
  Sustained rate **undecided**.
- **Related:** REQ-008 · EDGE-015
- **Blocked by:** Q-006 — "expected volume" is not a number.

### TEST-007 · `high` · boundary · **blocked**
**Event timestamped in the future**

- **Steps:** POST an event timestamped 2 hours ahead of server time.
- **Expected results:** **Undecided** — rejected, clamped, or accepted and flagged.
- **Related:** REQ-002 · EDGE-005
- **Blocked by:** Q-009.

### TEST-008 · `high` · recovery · **blocked**
**Alert delivery failure**

- **Steps:** Trigger an alert with a mailbox that hard-bounces and an unreachable
  number.
- **Expected results:** The failure is detected and visible somewhere a human looks.
  Retry and fallback behaviour **undecided**.
- **Related:** REQ-005 · EDGE-006
- **Blocked by:** Q-010. Note this test failing is indistinguishable from the system
  working, today — nothing observes delivery at all.

### TEST-009 · `high` · permission
**Ops replay does not page the customer**

- **Preconditions:** Device D with historical events that satisfied alert rule R.
- **Steps:** Ops replays D over that range.
- **Expected results:**
  1. Events are re-processed as requested.
  2. **No customer-facing alert is dispatched.**
  3. The replay is recorded with actor and range.
- **Related:** REQ-007 · FLOW-005 · EDGE-010
- **Note:** Expected to **fail** against a faithful implementation — REQ-007 states no
  boundary, and the obvious build re-fires alerts.

### TEST-010 · `high` · boundary
**Query spanning the retention edge**

- **Steps:** Query a 120-day range for a device with events throughout.
- **Expected results:** Events from the last 90 days returned, and the response states
  that earlier events are outside retention — not a silently truncated list.
- **Related:** REQ-006 · EDGE-014

---

## Identified but not written

| Would-be ID | Title | Type | Priority |
|---|---|---|---|
| TEST-011 | Alert storm — condition persists for 8 hours | timing | high |
| TEST-012 | Decommissioned device keeps posting | permission | medium |
| TEST-013 | Partial POST when battery dies mid-batch | network | medium |
| TEST-014 | Alert rule edited mid-evaluation | timing | medium |
| TEST-015 | Dashboard during in-progress backfill | timing | medium |

---

## Why 7 of 10 are blocked

A blocked test asserts only that *some* defined behaviour exists, never which one. A
test that invented an expected result would launder an assumption into a passing build
— the build would go green, and the green would mean nothing.

Two tests are expected to **fail** against a faithful implementation of this PRD:
`TEST-002` (unscoped API key) and `TEST-009` (replay pages customers). Those failures
are the findings.
