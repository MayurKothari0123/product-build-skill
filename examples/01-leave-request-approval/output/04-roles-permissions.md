> FlowBreaker · review · prd.md · 2026-08-15 (round 2)
> Status: complete

# Roles and Permissions

## Role matrix

| Role | Who they are | Their goal | In source? |
|---|---|---|---|
| Employee | Any staff member | Get leave approved without chasing; know where the request stands | yes (§2) |
| Manager | Line manager of one or more employees | Decide quickly without leaving the team short | yes (§2) |
| Skip-level manager | The manager's manager | Decide a manager's own request | **no — added by Q-001's answer** |
| HR Admin | People Ops | Accurate records; no manual reconciliation; policy compliance | yes (§1, §5) |
| Auditor | Compliance / external audit | Prove who approved what and when | **no — inferred from §5's "policy compliance"** |
| System | Scheduled jobs | Annual reset, escalation timers, notification delivery | **no — inferred from REQ-009** |

Three of six actors are absent from the source. The System actor matters more than
it looks: REQ-009's annual reset is the only requirement with no human trigger, and
nothing says what it does to requests that straddle the reset date (→ Q-010).

## Permission matrix

Role × action × resource. `undefined` cells are findings, never blanks.

| Action / Resource | Employee | Manager | Skip-level | HR Admin | Auditor |
|---|---|---|---|---|---|
| Submit request (own) | allow | allow | allow | allow | deny |
| View request (own) | allow | allow | allow | allow | — |
| View request (direct report's) | deny | allow | allow | allow | read-only |
| View request (anyone else's) | deny | **undefined → Q-016** | undefined | allow | read-only |
| Approve/reject (direct report's) | deny | allow | allow | **undefined → Q-014** | deny |
| Approve/reject (own) | deny | **deny** *(Q-001)* | deny | deny | deny |
| Amend request (own, pre-decision) | **undefined → Q-009** | undefined | undefined | undefined | deny |
| Cancel request (own, post-approval) | **undefined → Q-009** | undefined | undefined | undefined | deny |
| Reverse an approved request | deny | **undefined → Q-014** | undefined | **undefined → Q-014** | deny |
| View leave type on calendar | own only *(Q-004)* | direct reports | direct reports | allow | read-only |
| View availability on calendar | team | team | team | allow | read-only |
| Export monthly report | deny | **undefined → Q-012** | undefined | allow | **undefined → Q-012** |
| Adjust a balance manually | deny | deny | deny | **undefined → Q-017** | deny |

**7 undefined cells.** Each is a decision someone will make during implementation,
under time pressure, without the context to make it well — which is how permission
defects get built. Two are worth pulling out:

- **Reversing an approved request.** Leave gets cancelled, plans change, mistakes
  happen. There is no path back and no statement that there shouldn't be. Whoever
  implements this will either build an unaudited HR override or nothing at all.
- **Manual balance adjustment.** HR will need it — for corrections, carry-over,
  policy exceptions. Unspecified means unaudited by default, in the one place where
  an audit trail matters most.

## New permission rules created by answers

| Rule | Source | Note |
|---|---|---|
| A manager cannot approve their own request; it escalates to skip-level | Q-001 | Needs the fallback in Q-015 to be complete |
| Calendar visibility ≠ request visibility | Q-004 | The calendar cannot render the request object directly. This is an architectural constraint that arrived from a privacy answer, and it is easy to miss. |

## Data ownership

- **Owner of a leave request:** the submitting employee. *(inference — never stated)*
- **On deactivation:** pending requests auto-reassign to skip-level; the request
  record itself is retained. *(evidence — Q-002)*
- **Retention:** unspecified. Leave records include sick leave, which is
  health-adjacent personal data in most jurisdictions. No retention period, no
  deletion path, no statement of who may read historical records. → Q-018 (`high`)
