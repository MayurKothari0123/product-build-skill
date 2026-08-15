# FlowBreaker

**A requirements and UX stress-testing skill for AI coding agents.**

FlowBreaker doesn't write your PRD. It attacks it — finding the ambiguity, contradictions,
missing states, permission gaps and edge cases that turn into rework three sprints later. It
asks the questions your spec can't answer, and refuses to pretend an assumption is a fact.

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

> ### C-1 — Who approves? *(critical)*
> These cannot both describe the sole approver. Three readings are possible and the document
> supports none of them over the others. *Blocks: REQ-004, REQ-005, REQ-006, FLOW-002*
>
> **4 critical questions block flow analysis. Stopping here.**

It stops. It doesn't pick a reading and generate forty confident artifacts on top of a guess.

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

**→ [Open the full readiness report](https://htmlpreview.github.io/?https://github.com/MayurKothari0123/product-build-skill/blob/main/examples/01-leave-request-approval/output/report.html)**
— the actual generated `report.html`, rendered live.

Every artifact behind that verdict is committed here:
**[examples/01-leave-request-approval/](examples/01-leave-request-approval/)** — the input PRD,
and all twelve files FlowBreaker generated from it.

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

## Quickstart

```bash
curl -O https://raw.githubusercontent.com/MayurKothari0123/product-build-skill/main/SKILL.md
```

Then, in any agent:

```
Follow SKILL.md to review docs/prd.md
```

That's the whole installation. Two optional upgrades:

<details>
<summary><b>Make it trigger automatically</b> — so "review this PRD" just works</summary>

Add this to your repo's `AGENTS.md`. Claude Code, Codex and Cursor all read that file
automatically:

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

<details>
<summary><b>Install as a Claude Code skill</b> — gives you <code>/flowbreaker</code></summary>

`SKILL.md` ships with YAML frontmatter, so the same file doubles as a skill file. Other
agents ignore the frontmatter harmlessly.

```bash
mkdir -p .claude/skills/flowbreaker && cp SKILL.md .claude/skills/flowbreaker/SKILL.md
```
</details>

### Modes

| Mode | For | Runs |
|---|---|---|
| `quick` | A rough idea or small feature. ~10 min. | Restate → top 5 questions → likely edge cases |
| `review` | A real PRD. **Default.** ~20–30 min. | The full 9-step workflow below |
| `prototype` | A built prototype + approved requirements | Test plan, Playwright specs, findings |

```
Follow SKILL.md in quick mode: "add CSV export to the reports page"
Follow SKILL.md prototype mode against http://localhost:3000
```

**Input:** anything readable — a PRD, user stories, acceptance criteria, a Notion export, a
meeting transcript, three paragraphs in the chat. Structure helps but isn't required; *absence*
of structure is itself a finding it reports.

---

## How it works

Nine steps, with two gates that can halt the run.

| | Step | Produces |
|---|---|---|
| 1 | **Restate the problem** — in its own words, for you to confirm | `00-problem-brief.md` |
| 2 | **Audit the document** — contradictions, undefined terms, unfalsifiable requirements | `03-prd-audit.md` |
| 3 | **Generate and prioritize questions** | `02-questions.md` |
| | 🚦 **GATE 1** — if any question is `critical`, it **stops here** and asks | |
| 4 | **Roles and permissions** — who can do what, to whose data | `04-roles-permissions.md` |
| 5 | **Flow audit and state generation** — every state you didn't write | `05-flows.md` |
| 6 | **Edge cases** — with likelihood and impact | `06-edge-cases.md` |
| 7 | **Acceptance criteria review** — assessed, then rewritten testable | `07-acceptance-review.md` |
| 8 | **Test cases** — traceable and prioritized | `08-tests.md` |
| 9 | **Traceability and readiness** | `09-traceability.md`, `10-readiness.md` |
| | 🚦 **GATE 2** — the verdict is bound by rules, not vibes | `report.html` |

**GATE 1** is the difference between a review and a guess. Most tools fill an unanswered
question with a plausible assumption and keep going, so the output looks complete and quietly
rests on invented product decisions. FlowBreaker stops.

**GATE 2** binds the verdict to rules it can't talk itself out of: any open `critical`, or three
or more open `high` questions, and `PROCEED` is unavailable — no matter how good the rest looks.

### Answering questions

Reply in conversation ("Q-001: no, escalate to skip-level") and it writes back to
`02-questions.md`, or edit the `**Answer:**` line yourself. Statuses: `open`, `answered`,
`assumed`, `deferred`.

Say **"assume X and continue"** and it records `ASSUMP-00N` and proceeds — but it stays an
assumption in every artifact it touches and reappears in the readiness report. It never becomes
a fact. Say **"just carry on"** and it proceeds with questions open, and the verdict cannot be
`PROCEED`.

Answering one critical usually exposes the next layer down. Answering "a manager's own leave
escalates to skip-level" immediately raises: who approves for the CEO?

---

## Output

Written to `flowbreaker/` in your repo. Markdown so it diffs in a PR, HTML so it's readable by
people who don't live in a terminal.

| File | Contains |
|---|---|
| `report.html` | **The deliverable.** Self-contained, offline, light/dark. |
| `00-problem-brief.md` | The problem, restated. Jobs to be done. Non-goals. |
| `01-assumptions.md` | Gaps filled without an answer — and your document's own assumptions. |
| `02-questions.md` | The question queue. Where you write answers. |
| `03-prd-audit.md` | Requirements, clarity ratings, contradictions, undefined terms. |
| `04-roles-permissions.md` | Role and permission matrices. `undefined` cells are findings. |
| `05-flows.md` | Flow audit and the state matrix. |
| `06-edge-cases.md` | Edge-case register with likelihood and impact. |
| `07-acceptance-review.md` | Criteria assessed and rewritten. |
| `08-tests.md` | Test cases, traceable, prioritized. |
| `09-traceability.md` | Requirements↔flows↔tests, plus a reference-integrity check. |
| `10-readiness.md` | The readiness report in Markdown. |

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

**Questions are files, not chat scrollback.** `02-questions.md` is a queue with statuses, so a
review is resumable three weeks later by someone who wasn't in the original session. Chat
history isn't a work product.

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

**One file, no dependencies.** Portability across agents was worth more than features that would
require a runtime. It's plain Markdown, so it works anywhere and you can read the whole thing
before trusting it.

---

## Prototype mode

Needs approved requirements. Produces a test plan mapped to your flows, Playwright specs
(TypeScript) under `flowbreaker/prototype/specs/`, and a coverage report of what stays
unverifiable. Each spec names the `TEST-`, `REQ-` and `FLOW-` it verifies, so a failure points at
a requirement instead of a selector.

**FlowBreaker writes specs. You run them.** It reports what specs *would* verify, never what they
*did*. It will never authenticate or use credentials (including ones you offer), reach content
behind a login, bypass access controls, take irreversible external actions, or touch production.
Approval for one action isn't approval for the next. Paste results back and it marks those
requirements verified, with the run as evidence.

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
