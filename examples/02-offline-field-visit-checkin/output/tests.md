> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Test Cases

9 written at `critical` and `high` priority. **6 of 9 are blocked** — they cannot state
an expected result because nobody has decided what should happen.

Happy paths are deliberately deprioritised. This is the set of tests nobody would
otherwise write.

---

### TEST-001 · `critical` · timing · **blocked**
**Ops status while the technician is offline**

- **Preconditions:** Technician T assigned visit V. Device offline.
- **Steps:**
  1. T checks in at V, records work, checks out — all offline.
  2. Ops opens the visit list during that period.
- **Expected results:** **Undecided.** Ops sees *something* — but whether it is the
  stale assignment, an explicit "last synced" marker, or nothing at all is not stated.
- **Related:** REQ-006, REQ-007 · FLOW-004
- **Blocked by:** Q-001. The test asserts only that the state shown is not presented as
  current when it isn't.

### TEST-002 · `critical` · error · **blocked**
**Check-in with no GPS fix**

- **Preconditions:** Technician at a basement site. GPS unavailable.
- **Steps:** Attempt check-in.
- **Expected results:** **Undecided** — proceed with location unavailable, block, or
  use last known position.
- **Related:** REQ-002 · EDGE-002
- **Blocked by:** Q-003. Note that either answer makes something in the document wrong:
  blocking breaks §2's primary case, proceeding makes §5's confirmation optional.

### TEST-003 · `critical` · concurrency · **blocked**
**Same visit edited on device and in Ops**

- **Steps:**
  1. T edits visit V offline.
  2. Ops edits V in the console.
  3. T regains connectivity; V syncs.
- **Expected results:** **Undecided.** The test asserts only that no edit disappears
  without *someone* being told.
- **Related:** REQ-007 · EDGE-003 · ASSUMP-003
- **Blocked by:** Q-004.

### TEST-004 · `critical` · empty · **blocked**
**Technician starts the day offline**

- **Preconditions:** Fresh install or cleared cache. No connectivity.
- **Steps:** Open the app and view today's visits.
- **Expected results:** **Undecided** — cached assignments with a staleness marker, or
  an empty list, which makes the whole app unusable for that day.
- **Related:** REQ-001 · EDGE-001
- **Blocked by:** Q-008.

### TEST-005 · `high` · data · **blocked**
**Device clock manipulated**

- **Steps:** Set the device clock 3 hours back, check in, check out, sync.
- **Expected results:** **Undecided.** Recorded times drive billing, and the device
  clock is controlled by the person being billed for.
- **Related:** REQ-002, REQ-005, REQ-008 · EDGE-008
- **Blocked by:** Q-005. This is not primarily a fraud question — an incorrectly set
  clock produces the same result as a deliberately set one.

### TEST-006 · `high` · network · **blocked**
**Ten photos on a poor connection**

- **Steps:** Attach 10 photos offline; restore an intermittent 2G-grade connection.
- **Expected results:** **Undecided** — whether photos sync independently of the visit
  record determines whether the visit ever completes.
- **Related:** REQ-004 · EDGE-006 · ASSUMP-004
- **Blocked by:** Q-011.

### TEST-007 · `high` · recovery
**Visit never checked out**

- **Preconditions:** T checks in at V and the battery dies.
- **Steps:**
  1. Leave V open past end of day.
  2. Charge the device the next morning; open the app.
  3. Ops views V.
- **Expected results:**
  1. V is not silently discarded.
  2. V appears to Ops as an exception, not as still in progress overnight.
  3. T is prompted to close it with a recorded correction rather than editing history
     invisibly.
- **Related:** REQ-005 · EDGE-004, EDGE-016
- **Note:** Expected to **fail** against a faithful implementation — no requirement
  closes a visit that was never checked out. Paper handled this by being handed in
  regardless of completeness.

### TEST-008 · `critical` · boundary
**Billing export with unsynced visits outstanding**

- **Preconditions:** Period P contains 40 assigned visits; 3 remain on devices.
- **Steps:** Ops runs the billing export for P.
- **Expected results:**
  1. The export does **not** silently omit the 3.
  2. It states its coverage, or refuses to run, naming what is missing.
- **Related:** REQ-008 · EDGE-005
- **Note:** Expected to **fail** as specified. REQ-008 says "export completed visits";
  the natural implementation exports what the server has, which is the money-losing
  behaviour.

### TEST-009 · `high` · recovery
**Sync interrupted mid-batch**

- **Steps:** Queue 12 visits, begin sync, kill connectivity after 5, restore.
- **Expected results:** 12 visits server-side, not 17. The remaining 7 resume; the 5
  already uploaded are not duplicated.
- **Related:** REQ-007 · EDGE-011

---

## Identified but not written

| Would-be ID | Title | Type | Priority |
|---|---|---|---|
| TEST-010 | App update with unsynced visits in the old schema | data | critical |
| TEST-011 | Device lost with unsynced visits | permission | high |
| TEST-012 | Visit reassigned by dispatch after caching | timing | high |
| TEST-013 | Device storage full mid-visit | scale | medium |
| TEST-014 | Check-in at the wrong site | input | medium |

`TEST-010` is rated critical and unwritten because the behaviour it would assert does
not exist anywhere in the document — there is no local schema versioning to test.

---

## Why 6 of 9 are blocked

A blocked test asserts only that *some* defined behaviour exists, never which one.
Inventing an expected result would launder an assumption into a passing build.

Two tests are expected to **fail** against a faithful implementation: `TEST-007` (visit
never closed) and `TEST-008` (billing exports around missing data). Those failures are
the findings, and the second one is the one that costs money.
