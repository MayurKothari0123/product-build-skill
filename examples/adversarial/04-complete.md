# Spec: Saved Search Filters

**Author:** Search team · **Status:** Reviewed v2.0

> **Fixture note.** FlowBreaker must find **no critical questions** here. This spec
> is deliberately imperfect — there are real `medium` and `low` gaps — but nothing
> that blocks implementation. The correct output is a short review that reports the
> minor gaps honestly and says plainly that no critical gaps were found.

## 1. Problem

Support agents run the same ticket searches many times a day — "open tickets,
priority high, my team, last 7 days". They rebuild these filters by hand each time.
Instrumentation shows the median agent reconstructs an identical filter set 11 times
per shift, costing roughly 4 minutes per shift each. With 60 agents that is ~4 hours
of aggregate time per day.

## 2. Users

**Primary:** Support agents (60 today, ~100 by year end). Daily users of the ticket
console, comfortable with the existing filter UI, working on desktop.

**Secondary:** Team leads, who additionally want to share a standard filter set with
their team.

Agents currently work around this by bookmarking filter URLs. That breaks whenever a
filter parameter changes, and it cannot be shared without pasting links in chat.

## 3. Goal and non-goals

**Goal:** Let an agent save a filter combination, name it, and re-apply it in one
click.

**Non-goals for v1:**
- Sharing saved searches between users (v2 — team leads want it; deferred deliberately)
- Scheduled searches or digests
- Saved searches in any surface other than the ticket console
- Modifying the underlying filter UI

## 4. Requirements

**REQ-1 — Save a search.** An agent with an active filter set can save it with a
name. Names are 1–50 characters, unique per user, and trimmed of leading/trailing
whitespace. Saving stores the filter parameters, not the result set.

**REQ-2 — List saved searches.** Saved searches appear in a dropdown in the filter
bar, ordered by most recently used, then alphabetically. Maximum 20 per user; the
save control is disabled with an explanatory tooltip at the limit.

**REQ-3 — Apply a saved search.** Selecting one replaces the current filter set
entirely and runs the search. The applied search's name is shown in the filter bar
until the agent changes any filter, at which point it reverts to "unsaved".

**REQ-4 — Rename and delete.** An agent can rename (same validation as REQ-1) or
delete any of their own saved searches. Deletion asks for confirmation and cannot be
undone; the confirmation states the name being deleted.

**REQ-5 — Ownership.** Saved searches are private to the user who created them. No
user, including an admin, can view or apply another user's saved searches. On account
deactivation, saved searches are deleted with the account.

**REQ-6 — Invalid filters.** If a saved search references a filter value that no
longer exists (a deleted team, a removed priority level), applying it drops the
invalid clause, runs the remaining filters, and shows a non-blocking notice naming
what was dropped. The saved search is not modified.

## 5. States

| State | Behaviour |
|---|---|
| No saved searches yet | Dropdown shows "No saved searches. Save your current filters to get started." |
| Loading | Dropdown shows a spinner; the save control is disabled until loaded |
| Save fails | Inline error next to the name field, entered name preserved, retry available |
| Apply fails | Filter bar reverts to the previous filter set; toast with retry |
| At the 20 limit | Save control disabled, tooltip: "Delete a saved search to save a new one" |
| Duplicate name | Inline validation on blur, before submission is possible |

## 6. Permissions

| Action | Owner | Other agent | Team lead | Admin |
|---|---|---|---|---|
| Create own | allow | n/a | allow | allow |
| List own | allow | n/a | allow | allow |
| Apply own | allow | n/a | allow | allow |
| Rename/delete own | allow | n/a | allow | allow |
| View another's | deny | deny | deny | deny |
| Apply another's | deny | deny | deny | deny |

Authorization is enforced server-side on every endpoint; saved searches are scoped
by the authenticated user ID and never by a client-supplied parameter.

## 7. Accessibility

The dropdown follows the existing combobox pattern already used by the filter bar:
full keyboard operation, `aria-expanded` and `aria-activedescendant` maintained,
selection announced to screen readers. Delete confirmation traps focus and returns it
to the trigger on dismissal. No state is conveyed by colour alone.

## 8. Acceptance criteria

- Given an agent with an active filter set, when they save it with a valid name,
  then it appears at the top of their dropdown and persists across sessions.
- Given a name that duplicates an existing one, when the field loses focus, then
  inline validation blocks submission and states the conflict.
- Given an agent at 20 saved searches, when they open the save control, then it is
  disabled with the limit explained.
- Given a saved search referencing a deleted team, when applied, then the team clause
  is dropped, the rest runs, and a notice names the dropped clause.
- Given agent A's saved search ID, when agent B requests it directly via the API,
  then the server returns 404 (not 403, to avoid confirming existence).
- Given a delete confirmation, when confirmed, then the search is removed and the
  current filter set is unchanged.

## 9. Success metrics

- **Primary:** Median filter reconstructions per agent per shift drops from 11 to
  under 4 within 30 days of rollout.
- **Adoption:** ≥60% of agents have ≥1 saved search within 14 days.
- **Guardrail:** No increase in median ticket-console search latency (p50, p95).

## 10. Rollout

Behind a feature flag, enabled for one 12-agent team for one week, then all agents.
Rollback is flag-off; saved data is retained so re-enabling restores it.
