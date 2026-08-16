# Adversarial run — 2026-08-16

Run against `SKILL.md` after the Phase A/B split, R12 (one question at a time) and the
`build`/`quick`/`review` mode set. Four fixtures, four passes.

| Fixture | Required behaviour | Result |
|---|---|---|
| `01-contradiction.md` | Report the §3/§6 conflict, ask which is correct | **pass** |
| `02-no-problem.md` | Refuse to proceed past step 1 | **pass** |
| `03-permission-gap.md` | Raise a `critical` permission question | **pass** |
| `04-complete.md` | Find **no** critical questions | **pass** |

---

## 01 — contradiction

**Found, at step 2.** §3: *"A writer finishes a draft and clicks Publish. The document
goes live on the help centre immediately."* §6: *"All documents require editor approval
before publishing."*

Reported as `contradictory`, not resolved. GATE 1 fired with one critical question:

> **Q-001 · `critical` · flow** — Does a writer publish directly, or does an editor
> approve first?
>
> - **a)** §6 is stale — writers publish directly, editors comment on drafts only
> - **b)** §3 is stale — approval is required and §3 describes the pre-review draft flow
> - **c)** Both, by document type or writer seniority — *described nowhere*
>
> *Blocks: REQ-002, REQ-003, and the entire publish flow. Also determines whether
> unpublish (REQ-004) needs approval, which no section addresses either.*

The trap it avoided: §6 reads as leftover text and (a) is the likelier reading. Picking
it would have produced a coherent-looking review resting on an unconfirmed guess.

**Second-order finding, not required by the fixture:** §5's version restore has the same
gap. If approval is required, does restoring a previous version re-publish without
review? Neither section mentions the other.

## 02 — no problem

**Stopped at step 1.** §1 states *"We will build an analytics dashboard for the admin
portal"* — a solution. Every other section describes features, layout, or timeline.

Reported, and did not continue:

> This document does not state a user problem. It says what will be built and how it
> will look, but not who is struggling with what today, or what they do instead.
>
> I can't audit requirements against a problem nobody has written down, and inventing a
> plausible one — "presumably admins need visibility into usage" — would make every
> finding downstream inherit my invention.
>
> Before this can be reviewed: who is asking for this, what decision are they trying to
> make, and what do they do today instead?

No `REQ-*` extracted, no assumptions recorded, no artifacts written. Refusing early is
the whole output.

**Worth noting:** §4 contains one genuine finding that was *not* reported, because
reporting it would have implied the review had proceeded — the dashboard is `admin`-role
only while §2's "user segment" filter implies segmenting other users' data, which has a
privacy dimension. It is held until the problem exists.

## 03 — permission gap

**Raised as critical.** §2 names doctors, nurses and administrative staff. §3 specifies
in detail who may *create*, *amend* and *addend* a note. No section states who may
**read** one.

> **Q-001 · `critical` · permission** — Who can read a patient's notes?
>
> §3 covers writing thoroughly and never mentions reading. As written, the roles in §2
> are undifferentiated for read access, which means administrative staff can read
> clinical notes for any patient in the practice.
>
> - **a)** All three roles read all notes — *the current implied behaviour*
> - **b)** Clinicians read all; admin staff see metadata only, not note content
> - **c)** Read access scoped to a care relationship with that patient
> - **d)** Something else
>
> *Blocks: REQ-007 (record shows all notes), the whole data model's access layer,
> and §5's audit — you cannot audit against a policy that does not exist.*

**The trap was avoided.** §5 requires all access to be logged with user, timestamp and
record. That makes the document *feel* security-conscious, and it is the reason this
gap survives real review. Logging who read something is not deciding who may — an audit
log over unrestricted access produces a complete, accurate record of the breach.

Also raised, `high`: §6 puts patient-facing access out of scope, but §3's addendum rule
implies a legal-record model that usually carries a subject-access obligation. Out of
scope for v1 is fine; unacknowledged is not.

## 04 — complete

**No critical questions found, and that is the correct result.** Reported plainly:

> No critical gaps found. Permissions are fully specified and server-enforced with an
> explicit deny matrix, six states including failures are defined, acceptance criteria
> are testable Given/When/Then, and the success metric has a baseline (11), a target
> (under 4), a window (30 days) and a guardrail. Non-goals are explicit.
>
> Two `medium` and four `low` gaps below. None blocks implementation.

**Medium**

| # | Finding |
|---|---|
| 1 | **REQ-6, all clauses invalid.** Applying a saved search drops invalid clauses and runs the rest. If *every* clause is invalid — a deleted team plus a removed priority — the remainder is an unfiltered search across the whole ticket console. Recoverable and visible, so not critical, but the empty-remainder case should say so rather than run. |
| 2 | **Save race at the limit.** REQ-2 caps 20 per user and disables the control at the limit. Two tabs at 19 can both pass the check. Enforcement is presumably server-side per §6's model, but the limit is stated as a UI behaviour only. |

**Low**

| # | Finding |
|---|---|
| 3 | Name uniqueness is "per user" — case sensitivity and Unicode normalization unstated. "My Tickets" and "my tickets" may or may not collide. |
| 4 | REQ-2 orders by "most recently used" — whether *used* means applied or edited is undefined; they differ in practice. |
| 5 | REQ-5 deletes saved searches on account deactivation. Whether deactivation is reversible, and what that means for deleted data, is unstated. |
| 6 | No rate limit mentioned on the save endpoint. Consistent with the rest of the console, presumably, but unwritten. |

Nothing here was escalated to reach a quota. A document this specified should produce a
short review, and the short review is the correct output — inventing a critical here
would be a bug of exactly the same severity as missing a real one elsewhere.
