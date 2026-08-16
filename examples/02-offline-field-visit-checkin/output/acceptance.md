> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Acceptance Criteria

**The source contains none.** §8 offers three success criteria; none of them is testable
as written.

| REQ | Requirement | Testable | Unambiguous | States result | Covers failure | Verdict |
|---|---|---|---|---|---|---|
| REQ-001 | Technician sees assigned visits for the day | partly | no | no | **no** | rewrite |
| REQ-002 | Check-in records time and GPS location | partly | no | no | **no** | rewrite |
| REQ-003 | Records work performed — checklist plus notes | partly | no | no | no | tighten |
| REQ-004 | Technician can attach photos | partly | partly | no | no | rewrite |
| REQ-005 | Check-out records the time | partly | no | no | **no** | rewrite |
| REQ-006 | Ops sees visit status in near real time | **no** | **no** | no | no | **contradictory** |
| REQ-007 | Visit records sync automatically when connectivity returns | no | no | no | **no** | rewrite |
| REQ-008 | Ops can export completed visits for billing | no | **no** | no | no | rewrite |

## §8's success criteria, assessed

| Stated | Verdict | Why |
|---|---|---|
| "No more paper forms" | **measures adoption, not success** | Paper stops either because the app works or because it was mandated. Both look identical. |
| "Ops knows where technicians are" | **not served by any requirement** | Every requirement captures location once, at check-in. This criterion describes continuous tracking (Q-002). |
| "Billing goes out faster" | **unfalsifiable** | No baseline, no target, no definition of "out". |

---

## Rewrites

### REQ-006 — near real-time status
**Was:** "Ops sees visit status in near real time."
**Issues:** Directly contradicts §2 and §4. A technician working offline all day cannot
produce near-real-time status by any mechanism this document describes.
**Blocked by:** Q-001. Cannot be written until "near real time" is defined against
sync state.
**Shape it will need:**
- Given a technician with connectivity, when they check in, then the visit appears in
  Ops within N seconds.
- Given a technician offline, when Ops views the visit, then it shows its last known
  state **and the age of that state** — never a stale state presented as current.

### REQ-002 — check-in
**Was:** "Technician checks in at a site. Check-in records the time and GPS location."
**Issues:** No behaviour when GPS is unavailable — the primary case per §2. No
tolerance radius, so "on site" cannot be evaluated. Time comes from a device clock the
technician controls (Q-005).
**Suggested:**
- Given a technician within N metres of the site, when they check in, then the visit
  moves to `in-progress` with a server-verifiable timestamp and the coordinates.
- Given no GPS fix within N seconds, when they check in, then check-in **succeeds**,
  location is recorded as `unavailable` with a reason, and Ops sees that as a distinct
  state rather than a blank → **Q-003**.
- Given a device clock more than N minutes from server time, when the visit syncs, then
  the discrepancy is recorded and the visit is flagged for billing review.

### REQ-007 — sync
**Was:** "Visit records sync automatically when connectivity returns."
**Issues:** No conflict behaviour, no partial-batch behaviour, no failure surface. §4's
"the technician should not have to think about it" is a goal that becomes false the
moment a conflict exists.
**Suggested:**
- Given queued visits and restored connectivity, when sync runs, then each visit is
  uploaded exactly once and the queue count is visible to the technician.
- Given sync interrupted mid-batch, when connectivity returns again, then already
  uploaded visits are not duplicated and the remainder resume (EDGE-011).
- Given the same visit modified on device and in Ops, when it syncs, then — **undecided**
  → **Q-004**. Silent last-write-wins is the current implied behaviour and it deletes
  someone's work without telling either party.

### REQ-008 — billing export
**Was:** "Ops can export completed visits for billing."
**Issues:** "Completed" is undefined against sync state, which is the whole risk
(EDGE-005). No format, no period, no behaviour when data is missing.
**Suggested:**
- Given a period where every assigned visit has synced, when Ops exports, then all
  completed visits are included and the export states its coverage.
- Given any assigned visit in the period still unsynced, when Ops exports, then the
  export names the gap explicitly rather than silently omitting it → **Q-009**.

### REQ-004 — photos
**Was:** "Technicians can attach photos" (§6: up to 10 per visit).
**Issues:** Count is specified; resolution, total size, upload behaviour and failure
handling are not. Ten full-resolution photos on a rural connection is the case that
never completes.
**Suggested:**
- Given up to 10 photos, when attached offline, then they are stored locally at a
  compressed resolution and queued independently of the visit record, so a photo
  failure never blocks the visit from syncing (EDGE-006) → **Q-011**.

---

## Requirements with no acceptance criteria

**All eight.** The document describes what the technician *does* and almost never what
the system should do when connectivity, GPS, storage, battery or a second editor
interferes — which, per §2, is the normal operating condition.
