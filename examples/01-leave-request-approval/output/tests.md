> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Test Cases

Generated at `critical` and `high` priority only (§3 step 8). Lower-priority cases
are identified but not written — see the end of this file.

Several tests below assert *that defined behaviour exists* rather than which
behaviour, because the expected result is still an open question. Those are marked
**blocked**. A test that invents an expected result would launder an assumption into
a passing build (R3).

---

### TEST-011 · `critical` · permission
**Pending request whose approver is deactivated**

- **Preconditions:** Request R is `pending`, assigned to manager M. Employee E
  reports to M. M has a skip-level manager S.
- **Steps:**
  1. Deactivate M.
  2. Sign in as S; open the approval queue.
  3. Sign in as E; view request R.
  4. Sign in as HR; open the exception view.
- **Expected results:**
  1. R appears in S's queue.
  2. E sees an accurate status reflecting reassignment, not an unchanged "pending review".
  3. R appears in HR's exception view as reassigned.
  4. R is never in a state with no valid approver.
- **Related:** REQ-004, REQ-015 · FLOW-002 · EDGE-004 · Q-002

---

### TEST-012 · `critical` · concurrency
**Approving twice via browser back deducts the balance once**

- **Preconditions:** Employee E has balance 10. Request R for 3 annual days, `pending`.
- **Steps:**
  1. As manager, approve R.
  2. Press browser back.
  3. Press approve again.
- **Expected results:**
  1. Step 3 does not create a second approval.
  2. E's balance is 7, not 4.
  3. The manager sees the current decided state, not an actionable approve button.
  4. E receives exactly one notification.
- **Related:** REQ-004, REQ-006 · FLOW-002 · EDGE-007 · STATE-012
- **Note:** No requirement currently states that approval is idempotent. This test
  will fail against a straightforward implementation of the PRD as written — which
  is the point of writing it.

---

### TEST-010 · `critical` · permission
**Manager cannot approve their own request**

- **Preconditions:** Manager M reports to skip-level S. M submits request R.
- **Steps:**
  1. As M, open the approval queue.
  2. Attempt to approve R directly via its URL.
  3. As S, open the approval queue.
- **Expected results:**
  1. R does not appear in M's queue.
  2. Direct approval is rejected by the **server**, not merely hidden in the UI.
  3. R appears in S's queue.
- **Related:** REQ-015 · FLOW-002 · STATE-010 · Q-001
- **Note:** Step 2 is the test that matters. Hiding a button is not authorization,
  and a UI-only check is the most common way this requirement gets built wrong.

---

### TEST-006 · `high` · boundary
**Two pending requests cannot jointly overdraw the balance**

- **Preconditions:** Employee E has balance 5.
- **Steps:**
  1. Submit request A for 4 annual days. Leave `pending`.
  2. Submit request B for 4 annual days.
  3. Approve A, then approve B.
- **Expected results (blocked — EDGE-005, ASSUMP-006):**
  1. **Some** defined behaviour prevents the balance reaching −3.
  2. Whichever mechanism applies — blocking B at submission, re-checking at
     approval, or permitting a negative balance — it is deliberate and visible to
     both employee and approver.
- **Related:** REQ-002, REQ-006 · FLOW-001 · EDGE-005
- **Blocked by:** ASSUMP-006 is unvalidated. This test asserts that the case is
  handled at all.

---

### TEST-009 · `high` · boundary
**Past-dated requests**

- **Preconditions:** Employee E with balance.
- **Steps:**
  1. Submit annual leave for last week.
  2. Submit sick leave for last week.
- **Expected results (blocked — Q-006):**
  1. Behaviour is defined and consistent for both.
  2. If backdating is blocked, sick leave has an alternative path — retroactive sick
     leave is the normal case, not an exception.
- **Related:** REQ-001, REQ-012 · FLOW-001 · EDGE-001, EDGE-015

---

### TEST-013 · `high` · boundary
**Working-day counting across a weekend and a public holiday**

- **Preconditions:** Balance 10. Public holiday on the Monday.
- **Steps:** Submit annual leave Friday → following Tuesday. Approve.
- **Expected results (blocked — Q-006):**
  1. Days deducted match a stated definition of "working day".
  2. The employee saw the day count **before** submitting, not only after approval.
- **Related:** REQ-006 · FLOW-001 · EDGE-002

---

### TEST-019 · `high` · recovery
**A request nobody acts on**

- **Preconditions:** Request R `pending` for 14 days; no manager action.
- **Steps:** Advance time. Observe employee view, manager view, HR view.
- **Expected results (blocked — EDGE-012):**
  1. Some escalation, reminder, or visible ageing exists.
  2. The employee can see how long they have been waiting without asking anyone.
- **Related:** REQ-005, REQ-008 · FLOW-002 · EDGE-012
- **Note:** If this test cannot be written because no such behaviour is specified,
  that is the finding. This is the original problem (§1) reproduced inside the new
  system.

---

### TEST-021 · `high` · error
**Notification delivery failure**

- **Preconditions:** Manager M's email address bounces.
- **Steps:** Employee submits a request. Observe M's and the employee's views.
- **Expected results (blocked — Q-013):**
  1. The failure is detected rather than silently swallowed.
  2. The request is discoverable by M in-app, without relying on email.
  3. The employee is not told the manager was notified when they were not.
- **Related:** REQ-008 · FLOW-001, FLOW-002 · EDGE-016

---

### TEST-018 · `high` · concurrency
**Two approvers decide the same request simultaneously**

- **Preconditions:** R visible to both M and skip-level S.
- **Steps:** Both open R. M approves. S rejects one second later.
- **Expected results:**
  1. Exactly one decision is recorded.
  2. The loser sees the recorded decision and who made it.
  3. The employee receives exactly one notification.
  4. The balance reflects one decision.
- **Related:** REQ-004 · FLOW-002 · STATE-012 · EDGE-011

---

### TEST-007 · `high` · security
**Calendar does not leak leave type to peers**

- **Preconditions:** Employee E has approved sick leave. Peer P is on the same team,
  not E's manager.
- **Steps:**
  1. As P, open the team calendar.
  2. As P, request the calendar API directly and inspect the payload.
  3. As E's manager, open the calendar.
- **Expected results:**
  1. P sees "unavailable" with no type.
  2. **The API response contains no leave type for E** — not merely hidden in the UI.
  3. The manager sees the type.
- **Related:** REQ-007 · FLOW-004 · STATE-018 · Q-004
- **Note:** Step 2 is the real test. Filtering in the browser satisfies the design
  and still leaks the data.

---

### TEST-020 · `high` · error
**Employee with no manager assigned**

- **Preconditions:** New joiner N with no manager set.
- **Steps:** As N, submit a leave request.
- **Expected results (blocked — EDGE-013, Q-015):**
  1. Either submission is blocked with an actionable message, or it routes to a
     defined fallback.
  2. It does not silently create a request that reaches no queue.
- **Related:** REQ-003 · FLOW-001 · EDGE-013 · ASSUMP-003

---

## Identified but not written

Ask to generate any of these.

| Would-be ID | Title | Type | Priority |
|---|---|---|---|
| TEST-001 | Submit a valid request end to end | happy_path | medium |
| TEST-002 | Balance displays correctly on the form | happy_path | medium |
| TEST-003 | Manager sees only their team's requests | permission | medium |
| TEST-004 | Rejection without a reason is blocked | error | medium |
| TEST-005 | Employee notified on decision | happy_path | medium |
| TEST-008 | Manager notified on submission | happy_path | medium |
| TEST-014 | Request spanning the annual reset date | boundary | medium |
| TEST-015 | Balance service unavailable at submission | error | medium |
| TEST-016 | Form contents survive a failed submit | recovery | medium |
| TEST-017 | Overlapping requests from one employee | boundary | medium |
| TEST-022 | First-time empty state | empty | low |
| TEST-023 | Calendar loading vs. genuinely empty | empty | medium |
| TEST-024 | Keyboard-only completion of the submit flow | accessibility | medium |
| TEST-025 | Error messages associated with their fields | accessibility | medium |
| TEST-026 | Queue with 500+ pending requests | boundary | low |

**TEST-023 is under-rated at medium** and worth pulling forward: a calendar that is
loading and a calendar with nobody away render identically, and the wrong reading
causes someone to book leave into a clash. It is a one-line fix and a silent,
recurring failure if missed.

---

## Coverage

- Tests written: **11** (3 critical, 8 high)
- Tests identified, not written: **15**
- **Tests blocked on open questions: 6 of 11.** They assert that a defined behaviour
  exists rather than which one.
- Requirements with ≥1 test: **9 of 15** (60%)
- Requirements with no test: REQ-009, REQ-011, REQ-012, REQ-013, REQ-014, REQ-010

That six of eleven tests cannot state their expected result is the clearest single
measure of this specification's readiness. The tests are not the problem; they are
correctly reporting that nobody has decided yet.
