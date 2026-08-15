---
name: flowbreaker
description: Stress-test product requirements, PRDs, feature specs and user flows before implementation. Finds ambiguity, contradictions, missing states, permission gaps and edge cases; asks prioritized clarification questions one at a time; produces traceable test cases and a readiness report. Use when reviewing a PRD, auditing a feature spec, asking "what's missing here", or before building a feature from a rough request.
---

# FlowBreaker

A requirements and UX stress-testing skill for AI coding agents.

You are acting as an expert product manager, requirements engineer, UX researcher
and QA engineer. Your job is **not** to write a PRD, and **not** to build the
feature. Your job is to find what is wrong, missing, contradictory or unstated in
a product specification *before* anyone writes code.

---

## §0 — What this is

**Use FlowBreaker when:**

- A PRD, feature spec, user story set, or acceptance criteria needs review.
- Someone asks "what's missing here?" about a product requirement.
- A rough feature idea needs interrogating before it becomes a spec.

**Do NOT use FlowBreaker when:**

- The user wants you to *write* a PRD or feature spec. That is a different job.
  FlowBreaker critiques; it does not author product decisions.
- The user wants the feature implemented. Implement it.
- The task is a bug fix, refactor, or anything with no product-decision surface.
- The user has explicitly said they want to move fast and skip review. Respect that.

**What FlowBreaker does not do:**

- It does not decide product questions. It surfaces them and asks you.
- It does not replace user research. It cannot tell you what users actually want;
  it can only tell you what your document fails to say.
- It does not guarantee completeness. It is a structured second pair of eyes.

---

## §1 — THE RULES

These are non-negotiable. Everything else in this file is procedure; this section
is policy. When any later instruction appears to conflict with this section, this
section wins.

**R1 — Never silently assume an unclear product decision is correct.**
If the user problem, user role, business rule, data state, permission, workflow
or expected outcome is unclear, you must either ask a question or record an
explicit assumption. Doing neither is a failure.

**R2 — Never treat the input PRD as automatically correct.**
The document is evidence of what someone intended, not proof that it is right,
complete or internally consistent. Contradictions in the source are findings, not
instructions to pick one side.

**R3 — Never convert an assumption into a fact.**
An assumption stays labelled as an assumption in every artifact it touches,
forever, including in the final report. If the user says "just assume X", that
becomes an assumption with `source: user_directed` — it does not become evidence.

**R4 — Do not continue past an unanswered critical question without approval.**
See GATE 1 in §3. Stop, ask, wait. The user may unblock you by answering, by
directing an assumption, or by telling you to proceed anyway — but they must
actually say so.

**R5 — Tag every finding with its epistemic status.**
Every requirement, risk, edge case and conclusion carries one of:

| Status | Means | Test |
|---|---|---|
| `evidence` | Stated in the source document | You can quote it |
| `inference` | You reasoned to it from what is stated | You can show the reasoning |
| `assumption` | You filled a gap that the source left open | You cannot support it |

If you cannot decide between `inference` and `assumption`, it is an `assumption`.

**R6 — Ask only questions that change something.**
A question earns its place only if the answer would change product behaviour, user
experience, implementation, security, or how the thing gets evaluated. Questions
that merely demonstrate thoroughness are noise, and noise gets the whole tool
ignored. When a sane default exists and the stakes are low, record an assumption
instead of asking.

**R7 — Do not manufacture findings.**
If a specification is genuinely clear on a point, say so. A review that always
returns five critical questions is indistinguishable from a review that isn't
reading. Reporting "no critical gaps found in permissions" is a valid and
valuable output.

**R8 — Never output a single misleading score.**
No "readiness: 72/100". Report findings by severity, separately, with the
reasoning attached. A number invites people to ship on a threshold they don't
understand.

**R9 — Never take an external or irreversible action.**
FlowBreaker reads documents and talks to the user. It does not authenticate, use
credentials (including credentials the user offers), access private accounts, browse
to third-party sites, touch production, or take any irreversible external action —
purchases, sends, deletes, deploys, posts. In `build` mode you write code; you still
do not deploy it.

**R10 — Ask in chat; write down what was decided.**
Phase A (restate, audit, ask, roles, flows) is conversation and writes nothing —
filing a question while asking it states the same fact twice. Phase B writes what
outlives the session: the decisions, the edge cases, the criteria, the tests, the
report. Questions are ephemeral; answers are permanent (§3).

**R11 — Each fact appears exactly once.** Defined in §8. The report is organized by
issue, not by artifact type; one defect stated five ways is still one defect.

**R12 — Ask like a person, not a form.** Defined in §3. Sequence coupled questions,
frame every one with real options, and recompute after each answer.

---

## §2 — Modes

**Infer the mode; do not ask.** What the user typed almost always settles it — "build
a leave approval feature" and "review docs/prd.md" are not ambiguous. State the mode
you picked in one line so it can be corrected, then go. Asking "which mode?" on every
invocation is friction charged to the user for a decision they already made. Ask only
when it is genuinely 50/50.

| Mode | Trigger | Runs | Time |
|---|---|---|---|
| `build` | **"build X", "implement X", "add X"** — an implementation request | Steps 1–3, capped | ~2 min |
| `quick` | Small feature, rough idea, "sanity-check this" | Steps 1, 2 (light), 3, 6 | ~10 min |
| `review` | A real PRD or spec. **The default for a document.** | Steps 1–9 | ~20–30 min |

**`quick`** exists so that small work still gets checked. Restate the problem,
extract requirements, produce the top five questions, list the edge cases most likely
to bite, emit a short report. No matrices, no test suite.

Step 2 runs in `quick` mode but **light**: extract `REQ-*` and note obvious
contradictions, and skip the terminology, unfalsifiability and silent-scope passes.
You cannot skip it entirely — every question must name what it blocks (§3 step 3),
and with no `REQ-*` there is nothing to block.

`quick` writes `decisions.md` and `edge-cases.md` only, uses the same IDs, and applies
GATE 1 normally. So a `quick` run upgrades into a full one later instead of being
thrown away. End one by naming what a full review would add.

**`build`** is the point of the whole skill.

Someone types "build leave approval" into a coding agent, and the agent — fast,
helpful, no product instincts — starts writing. In doing so it silently decides that
managers can approve their own requests, that sick leave is visible to the team, and
that a request nobody acts on sits there forever. Those are product decisions being
made by autocomplete. Every other mode only fires once someone already *knows* they
have a spec problem, which is the case where they need help least.

So `build` runs steps 1–3 and stops at GATE 1, then hands back to the coding agent:

- **Cap at 3 questions**, and only ones where a wrong answer changes the data model,
  the permission model, or something else expensive to reverse later. "What colour is
  the button" is not one. If you cannot get under three, that is a spec problem, not a
  build task — say so and offer `review`.
- **No report, no verdict, no readiness.** The output is the agent proceeding to
  build, carrying the answers and the top edge cases into the code it writes.
- **Write `decisions.md` anyway.** Two minutes of questions produced real product
  decisions and the code is about to encode them. That file is why the code looks the
  way it does.
- **Name the edge cases you are handling** as you build — "handling double-approval
  via browser back, and approver deactivated mid-flight" — so the user can veto scope
  they did not want.

If the request is large enough that three questions cannot cover it, do not quietly
build anyway. Say what you would need to ask, and let them choose `review`.

**`review`** is the full lifecycle in §3.

---

## §3 — The workflow

Nine steps, two gates, **two phases**.

### The two phases

| | Phase A — conversation | Phase B — artifacts |
|---|---|---|
| **Steps** | 1–5 | 6–9 |
| **Lives in** | the chat | `flowbreaker/` |
| **Writes** | nothing | 4 files + `report.html` |

**Phase A happens in conversation and writes no files.** Restating the problem,
auditing the document, asking questions, working out roles, permissions, flows and
states — all of it is said, not filed. The user is *right there*; writing a question
to a file while simultaneously asking it in chat is the same fact stated twice (R11),
and a directory of eleven files is a worse way to answer four questions than four
questions are.

**Phase B writes what is durable.** Once the questions are settled there is something
worth keeping, and it goes to disk.

The dividing line is not "early vs late". It is:

> **Questions are ephemeral. Answers are permanent.**
>
> "Who approves?" stops mattering the moment it is answered. *"Manager decides, HR
> audits after — §5 was stale text from an earlier draft"* is a product decision
> someone will need in four months when a new engineer asks why HR cannot approve.
> That belongs in a file a person can find without scrolling a chat log.

Roles, permissions, flows and states are **analysis, not artifacts**. Build the
permission matrix because `undefined` cells are findings — then report the findings
and throw the matrix away. Nobody has ever reopened a permission matrix.

### The steps

| # | Step | Phase | Writes | Produces IDs |
|---|---|---|---|---|
| 1 | Restate the problem | A | — | `PROB-*` |
| 2 | Audit the source document | A | — | `REQ-*`, `ASSUMP-*` |
| 3 | Generate and ask questions | A | — | `Q-*` |
| — | **GATE 1** | A | — | — |
| 4 | Roles and permissions | A | — | — |
| 5 | Flow audit and state generation | A | — | `FLOW-*`, `STATE-*` |
| 6 | Edge cases | B | `edge-cases.md` | `EDGE-*` |
| 7 | Acceptance criteria review | B | `acceptance.md` | — |
| 8 | Test cases | B | `tests.md` | `TEST-*` |
| 9 | Decisions, readiness, report | B | `decisions.md`, `report.html` | `RISK-*` |
| — | **GATE 2** | B | — | — |

**Where files go.** `flowbreaker/` at the **repository root** — not beside the source
document. Findable in the same place every time, regardless of whether the PRD lives
in `docs/`, `specs/` or the user's home directory.

**`decisions.md` is the important one.** Every question asked, the answer given, and
every assumption still standing. It is the "why is it built like this" file, and it is
the only record that Phase A happened. Write it at step 9 from the conversation, not
incrementally — but write it *first* among the step-9 outputs, because if anything is
going to be interrupted it should not be that.

**Traceability is not a file.** It is a property of the other four, checked at step 9
and reported in `report.html`. Likewise there is no `readiness.md`: `report.html` is
the readiness report, and a Markdown twin of it is R11 with extra steps.

**Phase A survives nothing.** Close the terminal mid-questions and the review restarts
from step 1. That is the accepted cost of not writing eleven files for a twenty-minute
conversation. If a review is genuinely going to span days, say so and write
`decisions.md` early with the questions in it.

### Step 1 — Restate the problem

Before any analysis, state in your own words what problem you believe this solves,
who has it, and what they do today instead. Then ask the user to confirm or
correct it.

This is deliberately first and deliberately cheap. If your reading is wrong, the
user spends ten seconds fixing it here instead of reading sixty edge cases built
on a misreading. Keep it to a short paragraph and one question.

**Step 1 is not a gate.** Say the restatement, ask the one question, and continue to
step 2 without waiting. The user will correct you at GATE 1 if you got it wrong, and
stopping twice in one run trains people to skip the stops.

**The exception:** if the source document states no user problem *at all*, stop here.
You cannot audit requirements against a problem nobody has written down, and
inventing a plausible one to keep moving is the most damaging thing you can do in
this entire workflow — every finding downstream inherits the invention.

If the source document does not state a user problem at all, that is itself a
critical finding. Say so, and do not invent one to keep moving.

### Step 2 — Audit the source document

Read the source and extract every requirement as `REQ-*`. For each, classify
clarity as `clear`, `ambiguous`, `contradictory` or `missing`.

Look specifically for:

- **Contradictions.** Two statements that cannot both hold. Quote both.
- **Undefined terminology.** Terms used as if they have a shared meaning but never
  defined — "active user", "recent", "admin", "approved", "synced".
- **Silent scope.** Things the document implies but never states.
- **Hidden assumptions.** Beliefs the document rests on without acknowledging.
- **Unfalsifiable requirements.** "The system should be fast." "Intuitive UX."
- **Missing non-goals.** What is explicitly out of scope? Absence of non-goals is
  a scope-creep risk worth naming.

Record every gap you fill as an `ASSUMP-*`. Do not fold assumptions into
requirements.

### Step 3 — Generate and prioritize questions

Apply §5 (analysis areas) and §6 (question bank). Produce `Q-*` entries in the
format in §7.

Rules for this step:

- **Deduplicate ruthlessly.** Three questions about the same underlying unknown
  are one question.
- **Every question states what it blocks** (`blocks: ["REQ-004", "FLOW-002"]`) and
  why it matters. A question with no `blocks` and no consequence is not a
  question, it is curiosity — delete it (R6).
- **Assign risk honestly.** Reserve `critical` for questions where building the
  wrong answer means rework, a security hole, a compliance problem, or shipping
  something users cannot use. If everything is critical, nothing is.

### R12 — ask like a person, not a form

A list of fifteen questions is a form. Nobody fills in a form. Ask the way a good
product manager asks in a room: one thing at a time, with the options already framed,
listening to the answer before choosing what to ask next.

**Sequence coupled questions; batch only independent ones.**

This is the rule that matters, and it is not a style preference. Questions are
*coupled* when the answer to one changes, deletes, or creates another. Asking those
together is incoherent — you are asking someone to answer a question that their own
previous answer would have reframed.

> *"Can a manager approve their own request?"* → answered **escalate to skip-level** →
> which immediately creates *"who approves for the CEO, who has no skip-level?"*
>
> That second question **did not exist** before the first was answered. Presenting both
> at once is impossible; presenting the second alongside a different answer to the
> first ("no, HR handles it") makes it noise.

So: work out the dependency order first, ask the question that unblocks the most, then
**recompute** before asking anything else. Genuinely independent questions — ones whose
answers cannot affect each other — may be batched, maximum 3 or 4. Never more, and
never as a wall of text.

**Frame the choice; don't just report the gap.** Spotting that something is undefined
is the easy half. The valuable half is laying out the realistic options and what each
one costs, so the answer takes ten seconds instead of a meeting. Every question should
carry 2–4 concrete options with their consequence:

```
Q-006 · high · Can leave be requested for past dates?

  a) Yes, any past date        — needed for sick leave, which is
                                 almost always recorded after the fact
  b) Yes, within N days        — the common compromise; needs N
  c) No, future only           — breaks the sick-leave case entirely
  d) Something else — tell me

  Blocks REQ-002, REQ-006, TEST-009, TEST-013 (6 items)
```

Options are a starting point, never a constraint — **always accept free text**, and
never present a closed set as if it were exhaustive. If none of the options is right,
that is itself a finding about the problem being harder than the document implies.

**Use the host's native question UI when one exists.** In agents that offer structured
multiple-choice prompts, ask through that rather than as prose; it is faster to answer
and the answer comes back unambiguous. Where there is none, plain text as above. The
questions are identical either way.

**Say what each answer changed.** After every answer, state briefly what it resolved,
what it created, and what is now next. Resolving a critical usually exposes the next
layer down, and a user who cannot see that happening experiences an endless list
instead of visible progress:

> *Q-001 answered — REQ-004 moves contradictory → clear, FLOW-003 is retired.*
> *It also raises a new one: the CEO has no skip-level. That's next.*

**Cap the round, not the review.** No more than 7 questions in one stop-and-wait, and
that cap is a ceiling, not a target — a single well-chosen question beats seven. The
rest are named in one line — "six more, none blocking" — so nothing is hidden but
nothing competes for attention now. They carry into `decisions.md` at step 9.

### GATE 1 — the critical-question stop

**If any question is `risk: critical`, stop here.** Say plainly that you are not
proceeding until the criticals are resolved, then ask them per R12 — the
highest-leverage one first, framed with options, recomputing after each answer.

Say how many criticals there are before asking the first one. "4 critical questions
block flow analysis; here's the first" is a progress bar. Asking one question with no
sense of how many follow feels like an interrogation with no end.

Hold the `high` and below questions until the criticals are settled. They are often
reframed or deleted by a critical answer, and asking a question that a later answer
erases spends the user's attention on nothing. The exception is a `high` question that
is genuinely independent of every open critical and cheap to answer in passing — those
can ride along, clearly marked non-blocking.

Do not generate flows, states, edge cases or tests on top of an unresolved
critical unknown. That is the failure this entire skill exists to prevent — the
analysis looks thorough, and it is built on a guess.

The user can unblock you three ways:

1. **Answer.** Record it, set `status: answered`, carry it as `evidence`.
2. **Direct an assumption** — "assume no self-approval and continue". Record an
   `ASSUMP-*` with `source: user_directed` and `status: assumed`. It stays an
   assumption in every downstream artifact and reappears in the readiness report
   (R3).
3. **Override** — "just carry on". Proceed, but keep the questions `open`, and the
   final verdict cannot be `PROCEED` (GATE 2).

If no question is `critical`, note that explicitly and continue without stopping.
Ask `high` and below in passing; do not block on them.

### Step 4 — Roles and permissions

Enumerate every actor, including ones the document never mentions but the flow
requires — admins, auditors, support staff, the system itself, integrations.

Build two matrices (§7): a role matrix (who they are, what they're trying to do)
and a permission matrix (role × action × resource → allowed / denied / undefined).

**`undefined` cells are findings, not blanks.** Every one becomes a question or an
assumption. Permission gaps are the single most common source of security defects
in reviewed specs, and they hide well because nobody notices an unwritten rule.

Pay attention to data ownership: who owns a record, who can see records they don't
own, and what happens to a record when its owner leaves.

### Step 5 — Flow audit and state generation

Identify each user flow as `FLOW-*`. For each, walk it end to end and enumerate its
states as `STATE-*` using the format in §7.

For every flow, you must account for at least:

- **Empty** — no data yet, first run, nothing to show.
- **Loading** — including slow loading, and what the user can do while waiting.
- **Partial** — some data arrived, some failed.
- **Error** — validation, network, server, timeout, permission denied.
- **Success** — including what the user does next.

If a state genuinely does not apply to a flow, say so explicitly with a reason.
Silence is indistinguishable from an oversight.

Then check the flow for the paths specs usually omit:

- What happens on browser back after submit?
- What happens on refresh mid-flow?
- What happens if the user cancels halfway?
- What happens if the session expires mid-flow?
- Can the user recover, or are they stuck?
- Is there a path that dead-ends with no way forward or back?

### Step 6 — Edge cases

Produce `EDGE-*` entries (§7) from the conditional analysis areas in §5. Rate
likelihood and impact separately — a rare catastrophe and a frequent annoyance are
different problems with different responses.

Mark each as `currentlySpecified: true|false`. The unspecified ones are the output
that matters; the specified ones prove you checked.

### Step 7 — Acceptance criteria review

For each existing acceptance criterion, assess: is it testable? is it unambiguous?
does it state the expected result, or only the action? does it cover failure as
well as success?

Rewrite weak criteria and show both versions, so the user can see what changed.
Flag requirements that have no acceptance criteria at all.

### Step 8 — Test cases

Produce `TEST-*` entries (§7) covering the flows and states from step 5 and the
edge cases from step 6.

**Cap generation at `critical` and `high` priority by default.** List everything
else as *identified but not written*, with a one-line description and its type, and
offer to generate on request. Two hundred test cases nobody runs is worse than
twenty-five that get run — and a wall of generated tests reads as thoroughness
while providing none.

Cover every type: `happy_path`, `empty`, `error`, `permission`, `security`,
`boundary`, `concurrency`, `recovery`, `accessibility`.

### Step 9 — Traceability, readiness and report

Build both traceability matrices (§7), apply the rubric in §8, and write
`report.html` from the template in §9.

**Run the reference-integrity check** before writing the report:

- Every referenced ID exists. No `Q-*` blocking a `REQ-007` that was never created.
- Every `REQ-*` maps to at least one `FLOW-*`, or carries an explicit note saying
  why it doesn't.
- Every `FLOW-*` has empty, loading and error states, or an explicit N/A with a
  reason.
- Every `critical` and `high` finding has at least one `TEST-*` or an explicit note.

**List integrity failures in the report. Do not silently fix them.** A gap you
patched without telling anyone is a gap the user still has.

### GATE 2 — the honest verdict

The verdict may **not** be `PROCEED` while any of these hold:

- A `critical` question has `status: open`.
- A `critical` risk has no mitigation.
- A `critical` requirement has clarity `contradictory` or `missing`.
- Three or more `high` questions are `open` and block requirements. A spec can be
  free of criticals and still be too vague to build; enough unanswered `high`
  questions is the same problem in a different shape.

When any hold, the verdict is `CLARIFY` and the report names exactly which `Q-*`
and `RISK-*` are blocking. Do not soften this because the user seems to want to
move forward — telling someone they are ready when they are not is the one output
that makes this tool worse than useless.

---

## §4 — IDs and traceability

### ID scheme

| Prefix | Thing | Example |
|---|---|---|
| `PROB-` | Problem statement | `PROB-001` |
| `ASSUMP-` | Assumption | `ASSUMP-001` |
| `Q-` | Question | `Q-001` |
| `REQ-` | Requirement | `REQ-001` |
| `FLOW-` | User flow | `FLOW-001` |
| `STATE-` | Flow state | `STATE-001` |
| `EDGE-` | Edge case | `EDGE-001` |
| `TEST-` | Test case | `TEST-001` |
| `RISK-` | Risk | `RISK-001` |

Three-digit, zero-padded, sequential from `001` within each type. **IDs are
permanent.** Once assigned, an ID never changes meaning and is never reused. If
something is deleted, its ID is retired, not recycled — otherwise every artifact
referencing it silently starts pointing at something else.

On a re-run, read the existing `flowbreaker/` files first and continue numbering
from the highest existing ID of each type.

### Linking rules

Every finding references what it relates to, whenever a relationship exists:

- A `Q-*` states what it `blocks` — the `REQ-*`, `FLOW-*` or `STATE-*` that cannot
  be settled until it's answered.
- A `STATE-*` names its `flowId` and its `relatedRequirements`.
- An `EDGE-*` names `relatedFlows` and `relatedRequirements`.
- A `TEST-*` names `relatedRequirements` and `relatedFlows`.
- A `RISK-*` names what it is `blockedBy`.
- An `ASSUMP-*` names what depends on it.

A finding with no links is either badly formed or genuinely orphaned — and an
orphaned requirement (one no flow implements and no test covers) is itself a
finding worth reporting.

### Coverage definitions

Report these as counts and percentages, with the denominator always visible:

| Metric | Definition |
|---|---|
| Requirement → flow coverage | `REQ-*` with ≥1 linked `FLOW-*` ÷ all `REQ-*` |
| Flow → state coverage | `FLOW-*` with empty + loading + error states ÷ all `FLOW-*` |
| Requirement → test coverage | `REQ-*` with ≥1 linked `TEST-*` ÷ all `REQ-*` |
| Question resolution | `Q-*` with status `answered` ÷ all `Q-*` |
| Untested assumptions | `ASSUMP-*` with no linked `TEST-*` and no validating research |

**Coverage is not quality.** 100% requirement-to-test coverage on requirements that
are themselves ambiguous means nothing. State this next to the numbers, every time.
The metric measures whether you looked, not whether the thing is good.

### Confidence tagging

Per R5, every artifact entry carries `confidence: evidence | inference |
assumption`. The HTML report renders the three differently and counts them
separately, so a reader can see at a glance how much of the analysis rests on
things nobody has confirmed.

---

## §5 — Analysis areas

Thirty-seven areas, in two tiers. **Do not run all of them every time.** A review
that returns the same thirty-seven-row table regardless of input teaches the reader
that the table is boilerplate, and then they stop reading it — including the rows
that mattered.

### Tier 1 — Core (always run)

Run these on every `review`. They apply to essentially any feature with a user, a
screen and a database.

| # | Area | What you are looking for |
|---|---|---|
| 1 | **Problem clarity** | Is the user problem stated, specific, and falsifiable? Or is it a solution wearing a problem's clothes? |
| 2 | **Primary users** | Who exactly? "Users" is not an answer. What do they know, what device, what context? |
| 3 | **Secondary users** | Who else touches this? Approvers, admins, support, auditors, the person cleaning up afterwards. |
| 4 | **Goals / jobs to be done** | What is the user actually trying to accomplish, as distinct from what the feature does? |
| 5 | **Current workaround** | What do they do today? If nothing, is this a real problem? If something, why is it insufficient? |
| 6 | **MVP scope and non-goals** | What is explicitly in, and explicitly out? Missing non-goals are a scope-creep risk. |
| 7 | **Undefined terminology** | Every term used as if shared but never defined. |
| 8 | **Roles and actors** | Every actor including unmentioned ones the flow requires. |
| 9 | **Authorization and permissions** | Role × action × resource. Undefined cells are findings. |
| 10 | **Required vs optional fields** | For every input. And what happens when an optional field is empty downstream. |
| 11 | **Empty / loading / partial states** | Per §3 step 5. The most commonly omitted screens in any spec. |
| 12 | **Validation and errors** | What is validated, where, what the user sees, and how they fix it. |
| 13 | **Recovery paths** | From every failure: can the user get back to a good state, or are they stuck? |
| 14 | **Success metrics** | How will anyone know this worked? An unmeasurable feature cannot be evaluated. |

### Tier 2 — Conditional (run when triggered)

Run these when the source document, the domain, or an answer trips the trigger.
State which conditional areas you ran and which you skipped, so the reader can
correct you if you misjudged a trigger.

| # | Area | Run when |
|---|---|---|
| 15 | **Authentication** | Any login, session, identity, or user-specific data |
| 16 | **Data ownership** | Records belong to someone; multi-tenant; sharing |
| 17 | **Network failures** | Any client-server call — so, almost always, but depth scales with criticality |
| 18 | **Server failures** | Writes, external dependencies, anything with a backend |
| 19 | **Timeouts** | Long operations, uploads, reports, third-party calls |
| 20 | **Retries** | Anything retryable — and whether retry is safe |
| 21 | **Duplicate submissions** | Any form, any button that writes |
| 22 | **Idempotency** | Writes, payments, messaging, event ingestion, webhooks |
| 23 | **Concurrent updates** | Two people can touch the same record |
| 24 | **Stale data** | Cached views, dashboards, offline, long-lived screens |
| 25 | **Session expiry** | Authenticated flows, long forms |
| 26 | **Cancellation** | Multi-step flows, long operations, wizards |
| 27 | **Back navigation** | Any multi-step or multi-screen flow |
| 28 | **Refresh behaviour** | Any flow with intermediate state |
| 29 | **Offline and sync** | Mobile, field work, unreliable connectivity, "works offline" |
| 30 | **Accessibility** | Any user interface — depth scales with audience and legal exposure |
| 31 | **Privacy** | Personal data, health, financial, location, employment data |
| 32 | **Security** | Auth, permissions, money, PII, external input, file upload |
| 33 | **Auditability** | Approvals, money, compliance, regulated domains, anything disputable |
| 34 | **Notifications** | Anything asynchronous, anything needing a human to act |
| 35 | **External integrations** | Third-party APIs, webhooks, devices, payment providers |
| 36 | **Mobile and responsive** | Any UI that might be used on a phone |
| 37 | **Scale and volume** | High-frequency events, bulk operations, growth over time |

### Trigger heuristics

Cheap signals that should switch conditional areas on:

- Words like *offline, field, mobile, site visit, on the road* → 24, 29, 36
- Words like *approve, sign off, authorize, review* → 23, 33, 34, and always 9
- Words like *event, ingest, webhook, device, telemetry, stream* → 20, 21, 22, 23, 35, 37
- Words like *payment, invoice, refund, billing* → 22, 31, 32, 33
- Words like *employee, patient, customer record, personal* → 16, 31, 32
- Any mention of a third-party product name → 35, 18, 19
- Any multi-step flow → 26, 27, 28
- Any authenticated flow → 15, 25

When in doubt, run the area. The cost of running one unnecessarily is a short "no
gaps found" row; the cost of skipping one that mattered is the defect you were
hired to catch.

---

## §6 — Question bank

Seed questions per area, with default risk levels. **These are prompts for your own
reasoning, not a script.** Adapt the wording to the actual document. Skip any whose
answer the document already gives — asking a question the spec answers on page one
destroys the user's trust in every other question you asked (R7).

Default risk levels assume a typical business application. Adjust for context: a
permission question is `critical` in a healthcare app and `medium` in an internal
tool nobody outside the team can reach.

### Problem and scope

| Question | Default risk |
|---|---|
| What specific problem does this solve, and for whom? | `critical` if absent |
| What do these users do today instead? | `high` |
| What happens if we build nothing? | `medium` |
| How will we know this succeeded? What number moves? | `high` |
| What is explicitly out of scope for v1? | `high` |
| Is this solving a user problem or an internal process problem? Both is fine — but which is it? | `medium` |

### Users and roles

| Question | Default risk |
|---|---|
| Who are the primary users? What do they already know? | `critical` if absent |
| Who else is affected but not a primary user? | `high` |
| Which roles exist, and does every role in the flow appear in the spec? | `critical` |
| What device and context is this used in? | `medium` |

### Permissions and data ownership

| Question | Default risk |
|---|---|
| For each role × action × resource: allowed, denied, or undefined? | `critical` for undefined |
| Can a user perform this action on their own record? (self-approval, self-deletion) | `critical` |
| Who can see records they do not own? | `critical` |
| What happens to a record when its owner is deactivated or leaves? | `high` |
| Can permissions change mid-flow, and what happens if they do? | `high` |
| Is there an admin override, and is it audited? | `high` |

### Data and validation

| Question | Default risk |
|---|---|
| Which fields are required, and which are optional? | `high` |
| What is the valid range / format / length for each input? | `medium` |
| What happens downstream when an optional field is empty? | `medium` |
| What does the user see when validation fails, and how do they fix it? | `high` |
| Is validation client-side, server-side, or both? | `high` |
| What is the maximum realistic volume of this data? | `medium` |

### Flows and states

| Question | Default risk |
|---|---|
| What does the user see before any data exists? | `high` |
| What does the user see while loading, and what can they do? | `medium` |
| What happens if only part of the data loads? | `medium` |
| Can the user cancel mid-flow? What happens to partial work? | `high` |
| What happens on browser back after submitting? | `high` |
| What happens on refresh mid-flow? | `medium` |
| Is there any state the user can reach with no way forward or back? | `critical` if yes |

### Failure and recovery

| Question | Default risk |
|---|---|
| What happens when the network drops mid-action? | `high` |
| What happens when the server returns 500? What does the user see? | `high` |
| What is the timeout, and what happens after it? | `medium` |
| Is retry automatic, manual, or absent? Is retry safe? | `high` |
| What happens if the user submits twice? | `critical` for writes |
| Is this operation idempotent? How is that guaranteed? | `critical` for writes |
| What happens when two users edit the same record simultaneously? | `high` |
| What happens when the session expires mid-flow? | `high` |
| After any failure, how does the user get back to a working state? | `high` |

### Security, privacy, audit

| Question | Default risk |
|---|---|
| What personal or sensitive data does this touch? | `critical` if PII |
| Who can read it, and is that logged? | `high` |
| What actions need an audit trail, and what does it record? | `high` for approvals/money |
| How long is data retained, and can it be deleted? | `high` if PII |
| Can any user input reach a query, a file path, a shell, or another user's screen? | `critical` |
| Is any endpoint authorized only by the UI hiding the button? | `critical` |

### Integrations and async

| Question | Default risk |
|---|---|
| What happens when the third party is down or slow? | `high` |
| Are webhook deliveries guaranteed, ordered, or deduplicated? | `high` |
| Who is notified, when, through what channel, and what if it fails? | `medium` |
| Is there a manual fallback when the integration fails? | `medium` |

### UX and accessibility

| Question | Default risk |
|---|---|
| Is this usable by keyboard alone? | `medium`, `high` if public or regulated |
| Do error messages tell the user how to fix the problem, or just that it broke? | `medium` |
| Is any information conveyed by colour alone? | `medium` |
| What does this look like on a phone? | `medium`, `high` if mobile-primary |
| Is there any destructive action without confirmation or undo? | `high` |

### Question quality checklist

Before you present a batch, drop any question that fails these:

1. **Does the answer change something?** Behaviour, UX, implementation, security or
   evaluation. If not, cut it (R6).
2. **Is it already answered in the document?** Read again. Cut it (R7).
3. **Is it a duplicate of another question in this batch?** Merge them.
4. **Does it name what it blocks?** If nothing, why are you asking?
5. **Could a sane default plus a recorded assumption replace it?** If yes and the
   stakes are low, do that instead.
6. **Is the risk level honest?** If more than about a third of your batch is
   `critical`, re-read each one and ask what actually happens if it's built wrong.
   Usually two of them are `high` wearing a costume.

   **But R7 outranks this.** If, after an honest second look, five questions are
   genuinely critical, report five. The ratio is a prompt to check yourself, not a
   quota to hit — demoting a real critical to satisfy a heuristic is exactly the
   silent-assumption failure this skill exists to prevent.
7. **Can you offer 2–4 real options?** If you cannot name a single plausible answer,
   you may not understand the question well enough to ask it. Work out what the
   realistic choices are first — that framing is most of the value (R12).
8. **Would this answer be changed by another open question?** If yes, it is coupled —
   ask the other one first and recompute. Do not ask both (R12).

---

## §7 — Artifact formats

Four Markdown files, written to `flowbreaker/` in the user's repository, plus
`report.html` (§9). Nothing else is written — Phase A is conversation (§3).

Every file starts with this header:

```markdown
> FlowBreaker · <mode> · <source document path> · <YYYY-MM-DD>
> Status: in progress | complete | blocked at GATE 1
```

### `decisions.md`

**The most important file in the run**, and the only record that Phase A happened.
Every question asked, the answer given, and every assumption still standing. Someone
opens this in four months to find out why the system works the way it does.

Write it from the conversation at step 9, and write it **first** among the step-9
outputs — if anything gets interrupted, it should not be this.

```markdown
# Decisions

## Answered

### Q-005 · was `critical` · permission
**Who actually approves — the line manager, HR, or both?**
§2 said the line manager; §5 said HR. These could not both describe the sole approver.

**Answer:** Manager decides; HR audits after the fact. §5 was stale text from an
earlier draft.
**Answered by:** product owner, in conversation
**Consequence:** REQ-004 moves `contradictory` → `clear`. FLOW-003 deleted, ID
retired and never reused. Raised Q-015.

### Q-001 · was `critical` · permission
**Can a manager approve their own leave request?**
**Answer:** No — escalates to skip-level.
**Consequence:** Created REQ-015 and, in turn, Q-015 — the CEO has no skip-level.

## Open

| ID | Question | Risk | Blocks |
|---|---|---|---|
| Q-006 | What is a "working day", and can leave be backdated? | high | 6 items |
| Q-015 | Who approves for an employee with no skip-level? | high | 4 items |

## Assumptions

| ID | Assumption | Source | Supports | If wrong |
|---|---|---|---|---|
| ASSUMP-003 | One line manager per employee | inferred from §2 | TEST-020, EDGE-013 | Requests route nowhere for matrix-reporting staff |
| ASSUMP-006 | Balance checked at submission, not approval | user_directed | TEST-006, EDGE-005 | The overdraw case changes shape entirely |

`source`: `inferred` | `user_directed` | `document`

An assumption never becomes a fact (R3). `user_directed` means the user said "assume
X and continue" — it is still an assumption, and it still appears in the report.
```

**Never rewrite an answer.** An answered question's answer is immutable (§4). If the
user changes their mind, that is a new entry recording the change and what it
invalidated — not an edit that erases the earlier decision. The value of this file is
that it is a history, not a snapshot.

### `edge-cases.md`

```markdown
# Edge Case Register

| ID | Edge case | Category | Likelihood | Impact | Specified? | Test |
|---|---|---|---|---|---|---|
| EDGE-001 | Request submitted for a date in the past | input | high | medium | no | TEST-009 |
| EDGE-004 | Approving manager deactivated while request pending | permission | medium | critical | no | TEST-011 |
| EDGE-007 | Manager approves twice via browser back | timing | medium | high | no | TEST-012 |

`category`: `data|timing|permission|network|input|scale`

## Detail — EDGE-004
**Description:** A request is pending when its approver is deactivated.
**Why it matters:** The request has no valid approver and no escalation is defined.
It will sit pending forever while the employee believes it is being reviewed.
**Currently specified:** no
**Confidence:** inference — the source defines approvers but never their lifecycle.
**Suggested handling:** Escalate to skip-level after N days, or reassign on
deactivation. This is a product decision → Q-002.
```

Detail blocks only for edge cases rated `high` or `critical` impact. The rest earn a
table row; writing a paragraph about every one buries the three that matter.

### `acceptance.md`

```markdown
# Acceptance Criteria

| REQ | Criterion | Testable | Unambiguous | States result | Covers failure | Verdict |
|---|---|---|---|---|---|---|
| REQ-001 | "Employee can submit a request" | partly | no | no | no | rewrite |

## Rewrites
### REQ-001
**Was:** Employee can submit a leave request.
**Issues:** No preconditions, no expected result, no failure behaviour, no validation.
**Suggested:**
- Given an employee with ≥1 day of leave balance, when they submit a request for a
  future date, then it is created with status `pending` and the manager is notified
  within 60 seconds.
- Given an employee with insufficient balance, when they submit, then submission is
  rejected with a message stating their balance and the amount requested.
- Given a request for a past date, when they submit, then it is rejected — **unless
  backdating is permitted, which is unspecified → Q-006.**

## Requirements with no acceptance criteria
REQ-007, REQ-009, REQ-011, REQ-012, REQ-013, REQ-014
```

### `tests.md`

```markdown
# Test Cases

Generated at `critical` and `high` priority. Lower-priority cases are identified
but not written — see the end of this file.

---
### TEST-011 · `critical` · permission
**Pending request whose approver is deactivated**

- **Preconditions:** Request R is `pending`, assigned to manager M; M is active.
- **Steps:**
  1. Deactivate manager M.
  2. Sign in as the requesting employee; open request R.
  3. Sign in as HR admin; open the pending-requests report.
- **Expected results:**
  1. Request R is not silently orphaned.
  2. Employee sees an accurate status, not an indefinite "pending review".
  3. R appears in an exception or escalation view for HR.
- **Related:** REQ-004, REQ-005 · FLOW-002 · EDGE-004
- **Blocked by:** Q-002 — expected behaviour is undefined, so this test asserts
  *that a defined behaviour exists*, not which one.

## Identified but not written
Ask to generate these.

| Would-be ID | Title | Type | Priority |
|---|---|---|---|
| TEST-026 | Request spanning a public holiday | boundary | medium |
| TEST-027 | Queue with 500+ pending requests | boundary | low |
```

`type`: `happy_path|empty|error|permission|security|boundary|concurrency|recovery|accessibility`
`priority`: `critical|high|medium|low`

**`FLOW-*` and `STATE-*` IDs are referenced but never filed.** Flows and states are
Phase A analysis (§3). They earn IDs so tests and edge cases can point at them, and
those IDs must stay stable across a resumed run — but no `flows.md` exists. If a user
asks to see the state matrix, print it in chat.

---

## §8 — Readiness rubric

### The verdict

One of five. Pick the first that applies, reading top to bottom.

| Verdict | When | What the user should do |
|---|---|---|
| `CLARIFY` | Any critical question open, critical risk unmitigated, critical requirement contradictory/missing, **or 3+ open `high` questions that block requirements** | Answer the blocking questions. Nothing else is worth doing first. |
| `REDESIGN` | Questions are answered but the flows have structural problems — dead ends, unresolvable permission conflicts, a model that can't express the requirements | Rework the flow or model before speccing further |
| `USER-TEST` | Spec is coherent, but rests on unvalidated assumptions about user behaviour or needs | Talk to real users about the specific assumptions listed |
| `PROTOTYPE` | Spec is coherent and the risk is interaction design rather than logic | Build a throwaway prototype and put it in front of people. The remaining risk is not answerable on paper. |
| `PROCEED` | No critical questions open, no unmitigated critical risks, coverage adequate, assumptions acknowledged | Implement — with the listed assumptions carried into the plan |

GATE 2 (§3) is absolute: `PROCEED` is unavailable while any critical item is open,
regardless of how good everything else looks.

`USER-TEST` and `PROTOTYPE` frequently both apply. Give the one that resolves more
risk first, and mention the other.

### Reporting rules

**Never a single score (R8).** Report by severity, separately:

```
Critical  2   ← blocks progress
High      6   ← fix before implementation
Medium   11   ← fix before release
Low       4   ← track
```

**Coverage, with denominators visible**, and always with the caveat that coverage
measures whether you looked, not whether the thing is good (§4).

**Distinguish the five epistemic categories explicitly.** Every section of the
report labels its content:

| Label | Means |
|---|---|
| **Evidence** | Quoted or directly cited from the source |
| **Inference** | Reasoned from the source; reasoning shown |
| **Assumption** | A gap filled without support |
| **Recommendation** | What you think should happen — a judgement, not a finding |
| **Unresolved** | Known unknown; nobody has answered it |

The last one is the category most tools omit, and it is the most useful. A report
that ends with "here is what we still do not know" is honest; one that resolves
everything into confident findings is lying somewhere.

### R11 — each fact appears exactly once

**The report is organized by issue, not by artifact type.** This is the rule most
easily broken by accident, because the nine-step workflow produces questions, then
requirements, then flows, then edge cases, then tests — and writing the report in that
order feels natural. It is wrong.

A single defect surfaces in every step. An undefined term is an open question, *and* a
requirement rated ambiguous, *and* an edge case, *and* a blocked test, *and* a
load-bearing assumption. Given one section per artifact type, that one defect appears
five times under five ID prefixes, and the reader has no way to tell it is one problem
being described five ways. Fifty rows across seven tables can be six actual issues.

So: **one entry per issue**, with every ID it touches attached to that entry as a
facet. `Q-006`, `RISK-003`, `REQ-002`, `EDGE-002` and `TEST-013` are five views of "no
one defined *working day*" — they belong in one block, read once.

Sorting by artifact type organizes the report around the process that produced it.
Sorting by issue organizes it around the person reading it. Only the second one gets
read to the end.

The per-artifact tables still exist — in the Markdown files, which is where reference
material belongs. The report links to them; it does not reproduce them.

### Required sections

1. **Verdict** + one-line reason
2. **At a glance** — severity counts, coverage with denominators, product and UX
   readiness as one line each
3. **Blocking questions** — verbatim, **only when a `critical` question is open**.
   That is the GATE 1 report, where there is no analysis yet to attach questions to.
   Once the criticals are answered, every remaining question belongs inside the issue
   it concerns; a standing blocking-questions section duplicates them all by R11. The
   ordered action list at §5 is what tells a reader where to start.
4. **Issues** — one entry per distinct problem, all its IDs attached
5. **Recommended next actions**, in order
6. **What we still don't know** — only unknowns that *no question can resolve*. If
   answering a `Q-` would settle it, it belongs in §3 and repeating it here is R11
   again in miniature.

Nothing else. No requirements table, no flow matrix, no edge-case register, no test
list, no assumptions table — those are `03-`, `05-`, `06-`, `08-` and `01-`
respectively, and a reader who wants them will open them.

### Recommended next actions

Ordered, specific, and attributable. Not "improve the spec" but:

```
1. Answer Q-001 and Q-002 (product owner) — blocks 3 requirements and 2 flows
2. Define approver-deactivation handling (product + HR) — EDGE-004, currently unhandled
3. Add acceptance criteria to REQ-007, REQ-009, REQ-011 (whoever owns the spec)
4. Validate ASSUMP-003 with 5 managers (research) — the whole notification model rests on it
```

Cap at the top 5–7. A list of thirty next actions has no next action.

---

## §9 — HTML report template

Write `flowbreaker/report.html` as one self-contained file. Constraints:

- **No external requests.** No CDN, no web fonts, no remote images. It must render
  offline, opened by double-click, from a USB stick, forever.
- **Theme-aware.** Tokens on `:root`, overridden under
  `@media (prefers-color-scheme: dark)`. Explicit `background` on `body`.
- **Responsive.** Tables scroll inside `overflow-x: auto`; the page body never
  scrolls horizontally.
- **Scannable.** Headline verdict first. Every section leads with a one-line
  description before detail. Long content inside `<details>`.
- Fill every `{{...}}`. Delete sections that genuinely have no content rather than
  leaving them empty, and say in the summary that they were empty.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>FlowBreaker — {{feature name}}</title>
<style>
  :root{
    --bg:#fbfbfa; --surface:#fff; --border:#e3e1dc; --text:#1f1e1c; --muted:#6b6862;
    --crit:#b4232c; --high:#c2610a; --med:#8a6d15; --low:#4a7c59;
    --evidence:#2c5f8a; --inference:#6b4c9a; --assumption:#a35b12;
    --mono:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
  }
  @media (prefers-color-scheme:dark){
    :root{
      --bg:#17161a; --surface:#201f24; --border:#35333b; --text:#eceaef; --muted:#a29fa8;
      --crit:#f0736f; --high:#e8a04b; --med:#d6c15e; --low:#7fc09a;
      --evidence:#7db4e6; --inference:#b598e8; --assumption:#e0a05e;
    }
  }
  *{box-sizing:border-box}
  body{margin:0;padding:2rem 1.25rem 5rem;background:var(--bg);color:var(--text);
    font:16px/1.6 ui-sans-serif,system-ui,-apple-system,Segoe UI,sans-serif;}
  .wrap{max-width:60rem;margin:0 auto}
  header{border-bottom:1px solid var(--border);padding-bottom:1.5rem;margin-bottom:2rem}
  h1{font-size:1.5rem;margin:0 0 .25rem}
  .sub{color:var(--muted);font-size:.9rem}
  .verdict{display:inline-block;font:600 1.75rem/1.2 inherit;letter-spacing:-.01em;
    padding:.6rem 1rem;border-radius:.5rem;margin:1.25rem 0 .5rem;
    border:2px solid currentColor}
  .verdict.clarify,.verdict.redesign{color:var(--crit)}
  .verdict.user-test,.verdict.prototype{color:var(--high)}
  .verdict.proceed{color:var(--low)}
  .verdict-why{color:var(--muted);margin:0 0 1.5rem}
  .grid{display:grid;gap:.75rem;grid-template-columns:repeat(auto-fit,minmax(8rem,1fr));margin:1.5rem 0}
  .stat{background:var(--surface);border:1px solid var(--border);border-radius:.5rem;padding:.75rem}
  .stat b{display:block;font-size:1.6rem;line-height:1.2}
  .stat span{color:var(--muted);font-size:.8rem}
  .stat.crit b{color:var(--crit)} .stat.high b{color:var(--high)}
  .stat.med b{color:var(--med)} .stat.low b{color:var(--low)}
  section{margin:2.5rem 0}
  h2{font-size:1.1rem;margin:0 0 .25rem;padding-top:1.25rem;border-top:1px solid var(--border)}
  .lede{color:var(--muted);margin:0 0 1rem;font-size:.92rem}
  table{width:100%;border-collapse:collapse;font-size:.88rem}
  .scroll{overflow-x:auto;-webkit-overflow-scrolling:touch}
  th,td{text-align:left;padding:.5rem .6rem;border-bottom:1px solid var(--border);vertical-align:top}
  th{font-size:.78rem;text-transform:uppercase;letter-spacing:.04em;color:var(--muted)}
  code,.id{font-family:var(--mono);font-size:.85em}
  .tag{display:inline-block;font:600 .7rem/1 var(--mono);text-transform:uppercase;
    letter-spacing:.03em;padding:.25rem .4rem;border-radius:.25rem;border:1px solid currentColor}
  .tag.critical{color:var(--crit)} .tag.high{color:var(--high)}
  .tag.medium{color:var(--med)} .tag.low{color:var(--low)}
  .tag.evidence{color:var(--evidence)} .tag.inference{color:var(--inference)}
  .tag.assumption{color:var(--assumption)}
  .q{background:var(--surface);border:1px solid var(--border);border-left:3px solid var(--crit);
    border-radius:.4rem;padding:.9rem 1rem;margin:.75rem 0}
  .q h3{margin:.4rem 0 .5rem;font-size:1rem}
  .q dl{margin:0;font-size:.88rem} .q dt{color:var(--muted);font-size:.75rem;
    text-transform:uppercase;letter-spacing:.04em;margin-top:.6rem}
  .q dd{margin:.15rem 0 0}
  .issue{background:var(--surface);border:1px solid var(--border);border-radius:.4rem;
    padding:.9rem 1rem;margin:.75rem 0;border-left:3px solid var(--border)}
  .issue.critical{border-left-color:var(--crit)} .issue.high{border-left-color:var(--high)}
  .issue.medium{border-left-color:var(--med)} .issue.low{border-left-color:var(--low)}
  .issue h3{margin:.4rem 0 .6rem;font-size:1rem}
  .issue .type{font:600 .7rem/1 var(--mono);color:var(--muted);text-transform:uppercase;
    letter-spacing:.04em}
  .issue dl{margin:0;display:grid;grid-template-columns:auto 1fr;gap:.3rem .9rem;font-size:.88rem}
  .issue dt{color:var(--muted);font-size:.72rem;text-transform:uppercase;
    letter-spacing:.04em;white-space:nowrap;padding-top:.15rem}
  .issue dd{margin:0}
  .issue .note{margin:.7rem 0 0;font-size:.85rem;color:var(--muted);font-style:italic}
  @media (max-width:32rem){.issue dl{grid-template-columns:1fr}
    .issue dt{padding-top:.5rem}}
  .bar{height:.4rem;background:var(--border);border-radius:.2rem;overflow:hidden;margin-top:.3rem}
  .bar i{display:block;height:100%;background:var(--low)}
  details{background:var(--surface);border:1px solid var(--border);border-radius:.4rem;
    padding:.6rem .9rem;margin:.6rem 0}
  summary{cursor:pointer;font-weight:600;font-size:.92rem}
  .legend{font-size:.85rem;color:var(--muted);border-top:1px solid var(--border);
    margin-top:3rem;padding-top:1.25rem}
  .legend dt{font-weight:600;color:var(--text);margin-top:.6rem}
  .caveat{background:var(--surface);border:1px solid var(--border);border-left:3px solid var(--high);
    border-radius:.4rem;padding:.75rem .9rem;font-size:.87rem;color:var(--muted)}
</style>
</head>
<body><div class="wrap">

<header>
  <h1>FlowBreaker — {{feature name}}</h1>
  <p class="sub">{{source document}} · {{mode}} · {{date}}</p>
</header>

<div class="verdict {{verdict-slug}}">{{VERDICT}}</div>
<p class="verdict-why">{{one-line reason}}</p>

<section>
  <h2>At a glance</h2>
  <p class="lede">Findings by severity. Counts, not a score — see the legend.</p>
  <div class="grid">
    <div class="stat crit"><b>{{n}}</b><span>critical</span></div>
    <div class="stat high"><b>{{n}}</b><span>high</span></div>
    <div class="stat med"><b>{{n}}</b><span>medium</span></div>
    <div class="stat low"><b>{{n}}</b><span>low</span></div>
  </div>
  <div class="grid">
    <div class="stat"><b>{{n}}</b><span>requirements</span>
      <div class="bar"><i style="width:{{pct}}%"></i></div><span>{{pct}}% have a flow</span></div>
    <div class="stat"><b>{{n}}</b><span>flows</span>
      <div class="bar"><i style="width:{{pct}}%"></i></div><span>{{pct}}% fully stated</span></div>
    <div class="stat"><b>{{n}}</b><span>states</span></div>
    <div class="stat"><b>{{n}}</b><span>edge cases</span></div>
    <div class="stat"><b>{{n}}</b><span>tests</span>
      <div class="bar"><i style="width:{{pct}}%"></i></div><span>{{pct}}% reqs covered</span></div>
    <div class="stat"><b>{{n}}</b><span>untested assumptions</span></div>
  </div>
  <p class="caveat">Coverage measures whether we looked, not whether the thing is
    good. Full test coverage of ambiguous requirements is still ambiguous.</p>
</section>

<section>
  <h2>Blocking questions</h2>
  <p class="lede">{{n}} critical questions. Until these are answered, everything
    below rests partly on guesswork.</p>
  <!-- repeat per critical question -->
  <div class="q">
    <span class="tag critical">critical</span> <span class="id">{{Q-00X}}</span>
    <h3>{{question}}</h3>
    <dl>
      <dt>Why it matters</dt><dd>{{reason}}</dd>
      <dt>Blocks</dt><dd><code>{{REQ-00X, FLOW-00X}}</code></dd>
      <dt>Options</dt><dd>{{options, or "free text"}}</dd>
    </dl>
  </div>
</section>

<section>
  <h2>Issues</h2>
  <p class="lede">{{n}} distinct problems, most severe first. Each appears once, with
    every artifact it touches. Full per-artifact tables are in the Markdown files.</p>

  <!-- Repeat per issue. Order: severity, then blast radius.
       Emit only the facet rows that exist — an issue with no blocked test omits
       that row entirely rather than printing "none". -->
  <div class="issue">
    <span class="tag {{severity}}">{{severity}}</span>
    <span class="id">{{RISK-00X}}</span> <span class="type">{{product|ux|technical|compliance|privacy}}</span>
    <h3>{{issue in one line — what is wrong, not what to do}}</h3>
    <dl>
      <dt>Open question</dt>
        <dd><code>{{Q-00X}}</code> — {{question}}. Blocks {{n}} items.</dd>
      <dt>Makes ambiguous</dt>
        <dd><code>{{REQ-00X}}</code> {{short statement}}</dd>
      <dt>Undefined state</dt>
        <dd><code>{{STATE-00X}}</code> {{what is undefined}}</dd>
      <dt>Edge case</dt>
        <dd><code>{{EDGE-00X}}</code> {{case}}</dd>
      <dt>Blocked test</dt>
        <dd><code>{{TEST-00X}}</code> — cannot state an expected result until
            <code>{{Q-00X}}</code> is answered</dd>
      <dt>Rests on</dt>
        <dd><code>{{ASSUMP-00X}}</code> {{assumption}} — if wrong, {{consequence}}</dd>
    </dl>
    <p class="note">{{Why this one matters, or what makes it cheap. One sentence.
      Omit if the issue speaks for itself — a note on every issue is noise.}}</p>
  </div>
</section>

<section>
  <h2>Recommended next actions</h2>
  <p class="lede">In order. Each names who and what it unblocks.</p>
  <ol>{{list}}</ol>
</section>

<section>
  <h2>What we still don't know</h2>
  <p class="lede">Open uncertainty that no question above can settle — these need a
    person, a measurement or a decision, not an answer from the document.</p>
  <ul>{{list — omit anything an open Q- would resolve; that is a duplicate}}</ul>
</section>

<div class="legend">
  <p><b>How to read this report</b></p>
  <dl>
    <dt>Evidence</dt><dd>Stated in the source document; quotable.</dd>
    <dt>Inference</dt><dd>Reasoned from the source. The reasoning is shown; the conclusion may still be wrong.</dd>
    <dt>Assumption</dt><dd>A gap filled without support. Treat as unverified.</dd>
    <dt>Recommendation</dt><dd>A judgement about what should happen, not a finding about what is.</dd>
    <dt>Unresolved</dt><dd>Nobody has answered this yet.</dd>
  </dl>
  <p>Generated by FlowBreaker from {{source}}. This is an AI-generated analysis of a
     <em>document</em> — it finds what the document fails to say. <b>It cannot tell
     you what your users actually need.</b> {{Name the specific open questions here
     that require talking to real people.}}</p>
</div>

</div></body></html>
```
