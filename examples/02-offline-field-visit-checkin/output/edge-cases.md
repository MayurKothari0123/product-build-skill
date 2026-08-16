> FlowBreaker · review · prd.md · 2026-08-16
> Status: complete

# Edge Case Register

16 identified. **0 are specified in the source.** §4 acknowledges that offline happens
and says the technician "should not have to think about it" — which is a goal, not a
behaviour, and it is the entire offline specification.

| ID | Edge case | Category | Likelihood | Impact | Specified? | Test |
|---|---|---|---|---|---|---|
| EDGE-001 | Technician starts the day already offline, with no visit list | network | high | critical | no | TEST-004 |
| EDGE-002 | GPS unavailable indoors at check-in | data | high | critical | no | TEST-002 |
| EDGE-003 | Same visit edited on device and in Ops before sync | timing | medium | critical | no | TEST-003 |
| EDGE-004 | Technician never checks out | input | high | high | no | TEST-007 |
| EDGE-005 | Billing export runs while visits are still unsynced | timing | high | critical | no | TEST-008 |
| EDGE-006 | 10 photos, poor signal, upload never completes | network | high | high | no | TEST-006 |
| EDGE-007 | Visit edited after it has been billed | timing | medium | high | no | — |
| EDGE-008 | Device clock is wrong or deliberately changed | data | medium | high | no | TEST-005 |
| EDGE-009 | Device lost or stolen with unsynced visits | permission | medium | critical | no | — |
| EDGE-010 | Device storage full mid-visit | scale | medium | high | no | — |
| EDGE-011 | Sync interrupted halfway through a batch | network | high | high | no | TEST-009 |
| EDGE-012 | Visit reassigned by dispatch after the technician cached it | timing | medium | high | no | — |
| EDGE-013 | Technician checks in at the wrong site | input | medium | medium | no | — |
| EDGE-014 | Two technicians check in to the same visit | permission | low | medium | no | — |
| EDGE-015 | App updated while unsynced visits are stored in the old schema | data | medium | critical | no | — |
| EDGE-016 | Battery dies between check-in and check-out | network | high | high | no | TEST-007 |

`category`: `data|timing|permission|network|input|scale`

---

## Detail — EDGE-001 · offline at the start of the day

**Description:** A technician's first site is rural. They open the app with no
connectivity and have never loaded today's assignments.

**Why it matters:** REQ-001 — "technician sees their assigned visits for the day" — is
the first thing that happens and the only requirement with no offline story. Every
other requirement assumes a visit record already exists locally. The document specifies
how visits get *out* of the device and never how they get *in*.

**Currently specified:** no
**Confidence:** inference — §4 describes upload only; nothing describes prefetch.
**Suggested handling:** Assignments prefetch on the last successful connection, with
the app showing how stale the list is. Needs a decision on what happens when dispatch
reassigns afterwards (EDGE-012). → **Q-008**

---

## Detail — EDGE-002 · GPS unavailable indoors

**Description:** The technician arrives at a basement plant room and taps check in. No
GPS fix is available.

**Why it matters:** §5 exists to confirm the technician was on site. §2 names basements
and industrial buildings as the reason the app must work offline. The verification
mechanism fails precisely where the work happens, so it verifies attendance on easy
sites and not on hard ones. Blocking check-in makes the app useless in its primary
case; allowing it silently makes §5 decorative.

**Currently specified:** no
**Confidence:** evidence — §2 names the environments, §5 states the mechanism; the
conflict is that no section acknowledges both.
**Suggested handling:** Check-in proceeds and records location as unavailable with a
reason, surfaced to Ops as a distinct state rather than a missing value. → **Q-003**

---

## Detail — EDGE-005 · billing on unsynced visits

**Description:** Ops runs the billing export on Friday. Three technicians have visits
from Thursday still sitting on their phones.

**Why it matters:** This is the failure that costs money and is invisible when it
happens. REQ-008 says Ops can export completed visits for billing; nothing defines
"completed" against sync state. The old paper process failed loudly — a missing form
was a missing form. This one fails silently: the export succeeds, the invoice goes out,
and the work is discovered unbilled weeks later, or never.

**Currently specified:** no
**Confidence:** inference — REQ-007 and REQ-008 never reference each other.
**Suggested handling:** The export states its sync horizon and refuses to run — or
warns loudly — while any assigned visit for the period is unsynced. → **Q-009**

---

## Detail — EDGE-015 · app update with unsynced data

**Description:** The app auto-updates overnight. Visits stored in the previous local
schema have not yet synced.

**Why it matters:** Offline-first apps hold user data the server has never seen. An
update that changes local storage without migrating it destroys the only copy. This is
uncommon per release and unrecoverable when it happens — a technician's whole day, gone,
with no server-side backup to restore from.

**Currently specified:** no
**Confidence:** inference — no versioning or migration appears anywhere in the document.
**Suggested handling:** Local schema versioning with forward migration, and a hard rule
that an update never runs while unsynced records exist in an older schema.
