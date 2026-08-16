# FlowBreaker

**Finds the edge cases that break in production — while they're still in a document.**
*A requirements and UX stress-testing skill for AI coding agents.*

Not the ones your spec got wrong. The ones it never mentioned: the manager who approves their
own leave, the request nobody ever actions, the sick day that shows up on a public calendar. It
gives a coding agent the instincts of a product manager — the reflex to notice an undefined
term, an unwritten permission rule, a flow state nobody designed — and then to *ask* rather than
assume.

It doesn't write your PRD. It attacks it, one question at a time, and refuses to pretend an
assumption is a fact.

One Markdown file. Works with Claude Code, Codex, Cursor, Gemini CLI, Aider, or any agent that
reads files. No dependencies, no build step, no version to pin.

```
Follow SKILL.md to review docs/prd.md
```

---

## See it work

Here's a real line from a real leave-management PRD. It reads fine:

> **§2** — "The employee's line manager approves or rejects it."

And forty lines later, also fine:

> **§5** — "HR reviews and approves all leave requests to ensure policy compliance."

Two people wrote two sections. Both got reviewed. Nobody noticed. FlowBreaker's output:

> **4 critical questions block flow analysis. Stopping here — this is the first.**
>
> ### Q-005 · Who actually approves? *(critical)*
> §2 says the line manager. §5 says HR. These cannot both describe the sole approver,
> and the document supports no reading over the others.
>
> **a)** Manager decides, HR audits after — *fastest; HR sees problems late*
> **b)** HR approves after the manager — *two-stage, described nowhere else in the doc*
> **c)** §5 is stale text from an earlier draft — *most likely; needs confirming*
> **d)** Something else — tell me
>
> *Blocks REQ-004, REQ-005, REQ-006, FLOW-002*

It stops. It doesn't pick a reading and generate forty confident artifacts on top of a guess.

And it asks one at a time, because the answers are coupled — answering *"can a manager
approve their own request?"* with *"no, escalate to skip-level"* creates a question that
didn't exist a moment earlier:

> *Q-001 answered — REQ-004 moves contradictory → clear, FLOW-003 retired.*
> *It also raises a new one: the CEO has no skip-level. That's next.*

Answer the four, re-run, and it produces a verdict that argues with you:

> ## Verdict: `CLARIFY`
>
> **7 high-risk questions remain open, blocking 15 downstream items.** All four critical
> questions from round 1 are answered, which is real progress — the approval model is now
> coherent. But six of eleven test cases still cannot state an expected result, and a
> specification you cannot write a passing test against is not ready to implement.
>
> ```
> Critical  0   ← all resolved in round 1
> High     12   ← fix before implementation
> Medium   14   ← fix before release
> Low       4   ← track
> ```

**→ [Open the full readiness report](https://htmlpreview.github.io/?https://github.com/MayurKothari0123/product-build-skill/blob/main/report.html)**
— [`report.html`](report.html), exactly as FlowBreaker generated it.

**16 edge cases. 1 of them was in the source document** — and only because a question got
answered mid-review. The other fifteen were in nobody's spec, nobody's ticket and nobody's head,
and every one of them would have surfaced as a bug, a support thread, or a privacy incident.

Every artifact behind that verdict is committed here:
**[examples/01-leave-request-approval/](examples/01-leave-request-approval/)** — the input PRD,
and the four Markdown files FlowBreaker generated alongside the report.

Two more complete runs are committed:
**[offline field visit check-in](examples/02-offline-field-visit-checkin/output/)** (an app
required to work offline *and* report status in near real time — the document requires both and
reconciles neither) and
**[device event ingestion](examples/03-device-event-ingestion/output/)** (a one-minute alert SLA
promised for sensors the same document says go offline for hours). Both found **0 of their edge
cases specified in the source.**

---

## The problem it solves

Most specs are wrong in the same places. They detail the happy path and go quiet on everything
else. They say "the manager approves" without saying whether a manager can approve their own
request. They say "approved leave appears on the team calendar" without noticing that sick leave
is one of the types and the calendar is public. They say the process "should be fast" and call
that a requirement.

None of these are hard problems. They're *invisible* ones — the document reads fine, everyone
nods, and the gaps surface during implementation as small decisions made under time pressure by
whoever happened to hit them. FlowBreaker makes them visible while they're still cheap.

## Who it's for

| You are | You get |
|---|---|
| **PM or founder** writing specs | The questions a senior engineer asks in review, before the review |
| **Engineer** handed a vague ticket | A list of what to go ask, each item naming what it blocks |
| **Tech lead / architect** | Permission matrices and state coverage made explicit |
| **QA** | Edge cases and tests traceable back to specific requirements |
| **Designer** | Every flow state nobody specified: empty, loading, partial, error, denied |
| **Solo dev or small team** | A structured second pair of eyes that argues back |

You don't need to be technical. The output is Markdown plus a rendered HTML report, and the
questions are product questions, not code questions.

**Reach for it** before engineering estimates a spec · when a ticket says "should be intuitive"
and you need it testable · when inheriting a document whose author has left · after a feature
ships buggy, to find what class of gap the spec had · when turning a Notion page or meeting
transcript into something buildable · before a design review, to learn which states were never
designed.

---

## Using it

### 1 · Install — pick one

**As a Claude Code skill** (recommended — gives you `/flowbreaker` and automatic firing):

```bash
mkdir -p .claude/skills/flowbreaker
curl -o .claude/skills/flowbreaker/SKILL.md https://raw.githubusercontent.com/MayurKothari0123/product-build-skill/main/SKILL.md
```

**As a plain file** (Codex, Cursor, Aider, Gemini CLI — anything that reads files):

```bash
curl -O https://raw.githubusercontent.com/MayurKothari0123/product-build-skill/main/SKILL.md
```

That's the entire installation. No package manager, no build step, no version to pin.

<details>
<summary>Optional: make the plain-file install fire automatically too</summary>

Claude Code finds skills on its own. Codex and Cursor don't — they need to be told the file
exists. Add this to your repo's `AGENTS.md`, which both read automatically:

```markdown
## Product & requirements review
When asked to review a PRD, feature spec, user flow, or acceptance criteria —
or when asked "what's missing here" about a product requirement —
read ./SKILL.md and follow it exactly.
Do not skip its clarification-question gate.
```

Keep the pointer short and leave the skill in its own file — `AGENTS.md` loads into *every*
conversation in the repo, so inlining the skill costs tokens on every unrelated turn.
</details>

### 2 · Say what you want

You never name a mode. It reads your request and picks one, then says which in a line so you
can correct it.

| You type | Mode | What happens | Takes |
|---|---|---|---|
| *"build leave approval"* · *"implement CSV export"* · *"add SSO"* | `build` | Up to **3** questions, then it writes the code — with the edge cases handled | ~2 min |
| *"sanity-check adding a reports page"* · a rough idea in chat | `quick` | Restate → top 5 questions → the edge cases most likely to bite | ~10 min |
| *"review docs/prd.md"* · *"what's missing here?"* | `review` | The full 9-step workflow, both gates, full report | ~20–30 min |

With the skill installed you can also just type `/flowbreaker`. Without it, prefix anything with
*"Follow SKILL.md to…"*.

**`build` is the point of the whole thing.** Every other mode fires once you already know you
have a spec problem, which is when you need help least. `build` fires when someone types *"build
leave approval"* into a coding agent that's about to silently decide managers can approve their
own requests, that sick leave shows on the team calendar, and that a request nobody touches sits
there forever. Three questions maximum, and only where a wrong answer is expensive to reverse.
If it can't get under three, that's a spec problem — it says so and offers `review`.

### 3 · Answer the questions

It asks **one at a time**, with 2–4 options and what each one costs. You reply in chat like you
would to a colleague:

```
Q-001: no, escalate to skip-level
```

Or unblock it without answering — *"assume no self-approval and continue"* (recorded as an
assumption forever, never as a fact), or *"just carry on"* (proceeds, but the verdict can't be
`PROCEED`).

### What it takes as input

Anything readable — a PRD, user stories, acceptance criteria, a Notion export, a meeting
transcript, three paragraphs in the chat. Structure helps but isn't required; *absence* of
structure is itself a finding it reports.

---

## How it works

Nine steps, two gates, and **two phases**. Phase A is conversation and writes nothing. Phase B
writes what outlives the session.

| | Step | Phase | Produces |
|---|---|---|---|
| 1 | **Restate the problem** — in its own words, for you to confirm | A | — |
| 2 | **Audit the document** — contradictions, undefined terms, unfalsifiable requirements | A | — |
| 3 | **Ask questions** — one at a time, with options | A | — |
| | 🚦 **GATE 1** — if any question is `critical`, it **stops here** and asks | A | |
| 4 | **Roles and permissions** — who can do what, to whose data | A | — |
| 5 | **Flows and states** — every state you didn't write | A | — |
| 6 | **Edge cases** — with likelihood and impact | B | `edge-cases.md` |
| 7 | **Acceptance criteria** — assessed, then rewritten testable | B | `acceptance.md` |
| 8 | **Test cases** — traceable and prioritized | B | `tests.md` |
| 9 | **Decisions and readiness** | B | `decisions.md`, `report.html` |
| | 🚦 **GATE 2** — the verdict is bound by rules, not vibes | B | |

Steps 1–5 happen in chat because you're *right there* — filing a question to disk while
simultaneously asking it in chat is the same fact written twice, and a directory of eleven files
is a worse way to answer four questions than four questions are. Roles, permissions, flows and
states are analysis, not deliverables: the permission matrix exists so its `undefined` cells
become findings, and nobody has ever reopened one.

**GATE 1** is the difference between a review and a guess. Most tools fill an unanswered
question with a plausible assumption and keep going, so the output looks complete and quietly
rests on invented product decisions. FlowBreaker stops.

**GATE 2** binds the verdict to rules it can't talk itself out of: any open `critical`, or three
or more open `high` questions, and `PROCEED` is unavailable — no matter how good the rest looks.

Answering one critical usually exposes the next layer down — answering *"a manager's own leave
escalates to skip-level"* immediately raises *"so who approves for the CEO?"*. A falling
open-question count is not by itself a sign of progress.

---

## Output

Five files in `flowbreaker/`. Markdown so it diffs in a PR, HTML so it's readable by people who
don't live in a terminal.

| File | Contains |
|---|---|
| `report.html` | **The deliverable.** One entry per issue, self-contained, light/dark. |
| `decisions.md` | Every question, its answer, and every assumption still standing. The *why is it built like this* file. |
| `edge-cases.md` | Edge-case register with likelihood and impact. |
| `acceptance.md` | Criteria assessed and rewritten testable. |
| `tests.md` | Test cases, traceable, prioritized. |

There's no `readiness.md` — `report.html` **is** the readiness report, and a Markdown twin of it
would be the same content twice. Traceability isn't a file either; it's a property of the other
four, checked at step 9 and reported in the HTML.

Everything carries a stable, never-reused ID — `PROB-` `ASSUMP-` `Q-` `REQ-` `FLOW-` `STATE-`
`EDGE-` `TEST-` `RISK-` — and every finding links to what it relates to. A question names what
it blocks; a state names its flow; a test names both. That's what lets the report say *"Q-006
blocks 6 items across requirements, edge cases and tests"* instead of listing questions in a
pile.

---

## Design decisions

The parts that took the most thought, and why they are the way they are.

**It stops instead of guessing.** An unanswered critical question is a fork in the product, not
a blank to fill. Filling it produces a document that looks finished and is built on an invention
nobody agreed to — the exact failure this tool exists to catch.

**It asks one question at a time, with the options already framed.** A list of fifteen questions
is a form, and nobody fills in a form. More importantly, questions are usually *coupled* — the
answer to one deletes or creates another, so asking them together means asking someone to answer
a question their own previous answer would have reframed. Spotting that something is undefined
is the easy half; laying out the realistic options and what each one costs is the half that
turns a ten-minute meeting into a ten-second answer.

**Questions are ephemeral; answers are permanent.** "Who approves?" stops mattering the second
it's answered — but *"manager decides, HR audits after; §5 was stale text"* is a product decision
someone needs in four months when a new engineer asks why HR can't approve. So the asking happens
in chat and only the answers reach disk, in `decisions.md`. The cost is real and worth stating:
close the terminal mid-questions and the review restarts. That's the price of not writing eleven
files to answer four questions.

**Every claim is tagged `evidence`, `inference` or `assumption`** — and it's enforced in the
artifact schema rather than requested politely in a prompt, because blurring those three is
precisely the failure mode.

**Coverage prints its own caveat.** 87% requirement-to-flow coverage on ambiguous requirements
is 87% coverage of ambiguity. The number appears next to that sentence every time, because
coverage metrics are the easiest thing in the report to misread.

**`quick` mode exists so small work still gets checked.** A nine-step audit is right for a real
PRD and absurd for a CSV export button — and a tool people skip for small work stops being a
habit. It shares IDs and files with `review`, so a quick run upgrades later instead of being
thrown away.

**The report is organized by issue, not by artifact type.** One defect surfaces at every step of
the workflow — an undefined term is an open question, *and* an ambiguous requirement, *and* an
edge case, *and* a blocked test, *and* a load-bearing assumption. A section per artifact type
prints that one defect five times under five ID prefixes, and the reader can't tell it's one
problem. So the report gives each issue a single entry with every ID attached as a facet, and
the per-artifact tables stay in the Markdown files where reference material belongs. Sorting by
artifact organizes a report around the process that produced it; sorting by issue organizes it
around the person reading it.

**One file, no dependencies.** Portability across agents was worth more than features that would
require a runtime. It's plain Markdown, so it works anywhere and you can read the whole thing
before trusting it.

---

## How it's tested

There's no test runner — it's a prompt, not a program. It's verified against four
deliberately hostile fixtures in [`examples/adversarial/`](examples/adversarial/), each aimed at
one failure mode:

| Fixture | It must | Guards against |
|---|---|---|
| `01-contradiction.md` | Report the conflict and ask which is correct | Silently picking a side and building on it |
| `02-no-problem.md` | Refuse to proceed past step 1 | Inventing a user problem to keep moving |
| `03-permission-gap.md` | Raise a `critical` permission question | Treating an unwritten access rule as "presumably fine" |
| `04-complete.md` | **Find no critical questions** | Manufacturing findings to look thorough |

Last run: **[all four pass](examples/adversarial/RESULTS.md)** — including `04-complete`,
where the correct output is a short review reporting two `medium` and four `low` gaps
and saying plainly that nothing blocks implementation.

**`04-complete.md` is the important one.** Recall is easy — any sufficiently anxious reviewer
finds five criticals in any document. A tool that returns five criticals on every input teaches
its user to skim past all five, including the one that mattered. So inventing a critical for the
complete spec is a bug of exactly the same severity as missing a real one, and it's treated as
one.

---

## Limitations

Worth reading before you trust the output.

- **It analyses documents, not products.** A gap it can't see in the text, it can't report — and
  absence of a finding is not evidence of absence.
- **It will occasionally cry wolf.** Severity is calibrated for a generic business app. It can't
  tell whether "no audit trail" is critical (regulated company) or noise (tiny startup) unless
  you say so.
- **Inference can be confidently wrong.** It reasons from a document, having never used your
  product or met your users.
- **It has no idea what's technically expensive.** It may rate a week-long edge case the same as
  a one-line fix.
- **Two runs will differ.** It isn't deterministic.

**It cannot replace user research.** It can tell you your document never says who sees a
colleague's sick leave. It cannot tell you whether employees feel bad about that, whether they'd
stop using the system, or whether they've quietly worked around the current process for two
years. It reads what you wrote — not whether the problem you're solving is the one your users
actually have.

**A clean FlowBreaker report means your document is internally coherent. It does not mean you're
building the right thing.** Use it as a checklist that argues back, not an approval gate: the
verdict is an opinion with reasoning attached, and the reasoning is the valuable part.

---

## Contribution policy

This is a maintainer-controlled repository.

Public pull requests, issues, and discussions are currently not accepted. You may use, study,
and adapt FlowBreaker according to the Apache License 2.0.

Only explicitly authorized collaborators may modify the upstream repository.

For security concerns, contact: work.mayurbm@gmail.com

## License

Apache 2.0. See [LICENSE](LICENSE).
