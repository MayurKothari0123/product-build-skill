# FlowBreaker

**A requirements and UX stress-testing skill for AI coding agents.**

FlowBreaker does not write your PRD. It attacks it — finding the ambiguity,
contradictions, missing states, permission gaps and edge cases that turn into
rework three sprints later. It asks you the questions your spec can't answer, and
it refuses to pretend an assumption is a fact.

Works with Claude Code, Codex, Cursor, Gemini CLI, Aider, or any agent that can read
a Markdown file.

*(This repo is `product-build-skill`; the skill inside it is FlowBreaker.)*

---

## The problem it solves

Most specs are wrong in the same places.

They describe the happy path in detail and go quiet on everything else. They say
"the manager approves" without saying whether a manager can approve their own
request. They say "approved leave appears on the team calendar" without noticing
that sick leave is one of the types, and the calendar is visible to the whole team.
They say the process "should be fast" and call that a requirement.

None of these are hard problems. They're *invisible* problems — the document reads
fine, everyone nods, and the gaps surface during implementation as a stream of small
decisions made under time pressure by whoever happened to hit them.

FlowBreaker's job is to make those gaps visible while they're still cheap.

---

## What it does

- **Restates your problem back to you** before analysing anything, so a misreading
  costs ten seconds instead of a whole review.
- **Audits the document** for contradictions, undefined terms, unfalsifiable
  requirements and silent scope.
- **Asks prioritized questions** — and *stops* when a critical one is unanswered,
  instead of guessing and continuing.
- **Maps every flow state** you didn't write: empty, loading, partial, error,
  permission, concurrent, recovery.
- **Generates edge cases and test cases**, each traceable to a requirement.
- **Produces a readiness report** as a self-contained `report.html`, with findings
  separated by severity and every claim tagged as evidence, inference or assumption.

## What it does not do

- **It does not write your PRD.** It critiques; it doesn't author product decisions.
- **It does not decide anything.** Every product question it finds, it asks you.
- **It does not replace user research.** It reads documents. It cannot tell you what
  users want. See [Why real user research is still required](#why-real-user-research-is-still-required).
- **It does not guarantee completeness.** It's a structured second pair of eyes, not
  a proof.
- **It does not browse the web or drive your app.** Prototype mode writes Playwright
  specs; you run them.

---

## Installation

It's one Markdown file. That's the whole thing.

```bash
curl -O https://raw.githubusercontent.com/MayurKothari0123/product-build-skill/main/FLOWBREAKER.md
```

Then, in any agent:

```
Follow FLOWBREAKER.md to review docs/prd.md
```

No package manager, no manifest, no build step, no version to pin.

### Optional: make it trigger automatically

Add these six lines to your repo's `AGENTS.md` (create it if it doesn't exist).
Claude Code, Codex and Cursor all read this file automatically:

```markdown
## Product & requirements review
When asked to review a PRD, feature spec, user flow, or acceptance criteria —
or when asked "what's missing here" about a product requirement —
read ./FLOWBREAKER.md and follow it exactly.
Do not skip its clarification-question gate.
```

Now "review this PRD" works without naming the file. **Keep the pointer short and
leave the skill in its own file** — `AGENTS.md` loads into *every* conversation in
the repo, so putting the full skill there costs tokens on every unrelated turn.

### Optional: as a Claude Code skill

`FLOWBREAKER.md` ships with YAML frontmatter, so it doubles as a Claude Code skill:

```bash
mkdir -p .claude/skills/flowbreaker
cp FLOWBREAKER.md .claude/skills/flowbreaker/SKILL.md
```

Gives you `/flowbreaker` plus automatic invocation on matching tasks. Other agents
ignore the frontmatter harmlessly, so it's the same file either way.

---

## Usage

### Claude Code

```
/flowbreaker                                    # if installed as a skill
Follow FLOWBREAKER.md to review docs/prd.md     # always works
```

### Codex, Cursor, Aider, Gemini CLI

```
Follow FLOWBREAKER.md to review docs/prd.md.
Stop and ask me the critical questions before generating flows.
```

### Modes

| Mode | For | Runs |
|---|---|---|
| `quick` | A rough idea or small feature. ~10 min. | Restate → top 5 questions → likely edge cases |
| `review` | A real PRD. **Default.** ~20–30 min. | The full 9-step lifecycle |
| `prototype` | A built prototype + approved requirements | Test plan, Playwright specs, findings |

```
Follow FLOWBREAKER.md in quick mode: "add CSV export to the reports page"
Follow FLOWBREAKER.md prototype mode against http://localhost:3000
```

`quick` exists so small work still gets checked. A nine-step audit is right for a
real PRD and absurd for a CSV export button — and a tool people skip for small work
stops being a habit. Same IDs and files, so a `quick` run upgrades into a full one
later instead of being thrown away.

---

## Input format

Anything readable. A PRD, a feature spec, user stories, acceptance criteria, a
Notion export, a meeting transcript, or three paragraphs in the chat.

Markdown is easiest to quote back at you. Structure helps but isn't required —
FlowBreaker is designed for messy real documents, and *absence* of structure is
itself a finding it will report.

See [`examples/01-leave-request-approval/prd.md`](examples/01-leave-request-approval/prd.md)
for a realistic input: a plausible v0.3 draft with an approval contradiction, two
unfalsifiable success criteria, and several silent gaps.

---

## Output artifacts

Written to `flowbreaker/` in your repo:

| File | Contains |
|---|---|
| `report.html` | **The deliverable.** Self-contained, offline, light/dark. |
| `00-problem-brief.md` | The problem, restated. Jobs to be done. Non-goals. |
| `01-assumptions.md` | Every gap filled without an answer — and the assumptions your *document* makes. |
| `02-questions.md` | The question queue. Where you write answers. |
| `03-prd-audit.md` | Requirements, clarity ratings, contradictions, undefined terms. |
| `04-roles-permissions.md` | Role matrix and permission matrix. `undefined` cells are findings. |
| `05-flows.md` | Flow audit and the state matrix. |
| `06-edge-cases.md` | Edge-case register with likelihood and impact. |
| `07-acceptance-review.md` | Criteria assessed and rewritten. |
| `08-tests.md` | Test cases, traceable, prioritized. |
| `09-traceability.md` | Requirements↔flows↔tests, plus a reference-integrity check. |
| `10-readiness.md` | The readiness report in Markdown. |

Markdown so it diffs in a PR; HTML so it's readable by people who don't live in a
terminal.

A complete worked set is in
[`examples/01-leave-request-approval/output/`](examples/01-leave-request-approval/output/) —
open `report.html` in a browser.

---

## Example workflow

**Minute 0.** `curl -O …/FLOWBREAKER.md`

**Minute 1.** `Follow FLOWBREAKER.md to review docs/leave-approval-prd.md`

**Minutes 1–3.** It restates the problem and asks you to confirm:

> **PROB-001:** Leave is requested over WhatsApp, so there's no system of record and
> HR rebuilds it by hand each month.
> Is that the problem — or is the real problem employees not knowing their status?

**Minutes 3–8.** It audits the document, then **stops**:

> **4 critical questions block flow analysis.**
>
> **Q-005** Who actually approves — the line manager, or HR? §2 says manager, §5 says
> HR. These cannot both be the sole approver.
> *Blocks: REQ-004, REQ-005, REQ-006, FLOW-002*
>
> **Q-001** Can a manager approve their own leave request?
> **Q-002** What happens to pending requests when the approving manager leaves?
> **Q-004** Does the team calendar show leave *type*? Sick leave is one of them.

**Minutes 8–12.** You answer. Answering Q-001 ("escalate to skip-level") creates a
*new* question: who approves for the CEO? Resolving a critical usually exposes the
next layer down.

**Minutes 12–20.** Flows, states, edge cases, tests, traceability.

**Minute 20.** `flowbreaker/report.html`:

> ### CLARIFY — 7 high-risk questions remain open
> 15 requirements · 4 flows · 23 states · 16 edge cases · 11 tests
> **45% of tests can state an expected result** — the rest assert only that *some*
> behaviour is defined.

**Next week.** Your PM answers two more. Re-run: answered questions freeze, dependent
artifacts get flagged for revision, the verdict moves.

---

## Answering questions

Two ways:

1. **In conversation** — "Q-001: no, escalate to skip-level." FlowBreaker writes it
   back to `02-questions.md`.
2. **In the file** — edit the `**Answer:**` line and set `status: answered`.

Four statuses: `open`, `answered`, `assumed`, `deferred`.

You can also say **"assume X and continue"**. FlowBreaker records `ASSUMP-00N` with
`source: user_directed` and proceeds — but it stays an assumption in every artifact
it touches and reappears in the readiness report. It never becomes a fact.

Or **"just carry on"**. It proceeds with the questions still open, and the verdict
cannot be `PROCEED`.

**Questions are files, not chat scrollback.** That's what makes a review resumable
three weeks later by someone who wasn't in the original session.

---

## Traceability model

Stable IDs, permanent, never reused:

`PROB-` · `ASSUMP-` · `Q-` · `REQ-` · `FLOW-` · `STATE-` · `EDGE-` · `TEST-` · `RISK-`

Every finding links to what it relates to. A question names what it blocks. A state
names its flow and requirements. A test names both. This is what lets the report say
*"Q-006 blocks 6 items across requirements, edge cases and tests"* instead of listing
questions in a pile.

Coverage is reported with denominators visible:

| Metric | Definition |
|---|---|
| Requirement → flow | `REQ` with ≥1 `FLOW` ÷ all `REQ` |
| Flow → state | `FLOW` with empty + loading + error ÷ all `FLOW` |
| Requirement → test | `REQ` with ≥1 `TEST` ÷ all `REQ` |
| Question resolution | `Q` answered ÷ all `Q` |

**Coverage measures whether we looked, not whether the thing is good.** 87%
requirement-to-flow coverage on ambiguous requirements is 87% coverage of ambiguity.
FlowBreaker prints that caveat next to the numbers every time, because coverage
metrics are the easiest thing in this report to misread.

### Evidence, inference, assumption

Every finding is tagged:

| Tag | Means |
|---|---|
| `evidence` | Stated in your document. Quotable. |
| `inference` | Reasoned from it. Reasoning shown; conclusion may still be wrong. |
| `assumption` | A gap filled without support. Unverified. |

Blurring these is the failure this tool exists to prevent, so it's enforced in the
schema rather than requested politely.

---

## Prototype review mode

Needs approved requirements — an existing `flowbreaker/` or requirements you supply.

Produces a test plan mapped to your flows, Playwright specs (TypeScript) under
`flowbreaker/prototype/specs/`, a static findings pass over prototype source if you
point at it, and a coverage report of what remains unverifiable.

Each spec names the `TEST-`, `REQ-` and `FLOW-` it verifies, so a failure points at a
requirement instead of a selector. Where expected behaviour is still an open
question, the spec says so and asserts the weaker property — a test that invents an
expected result would launder an assumption into a passing build.

### Browser safety

**FlowBreaker writes specs. You run them.** It reports what the specs *would* verify,
never what they *did*.

It will never:

- Authenticate, log in, or use credentials — including credentials you offer it
- Access private accounts or content behind a login
- Scrape restricted content or bypass access controls
- Take irreversible external actions: purchases, sends, deletes, deploys, posts
- Touch a production system
- Submit real personal data to any form

Approval for one action isn't approval for the next. "Check localhost:3000" does not
authorize following a link off-site.

If you paste test results back, FlowBreaker incorporates them and marks those
requirements verified — with the run as evidence.

---

## Limitations of AI-generated analysis

Worth reading before you trust any of the output.

- **It analyses documents, not products.** Everything it knows comes from what you
  gave it. A gap it can't see in the text, it can't report.
- **It will miss things.** Absence of a finding is not evidence of absence.
- **It will occasionally cry wolf.** A question rated `critical` may be trivial in
  your context. Rating is calibrated for a generic business app; your domain differs.
- **Inference can be confidently wrong.** Findings tagged `inference` are reasoning
  from a document by something that has never used your product or met your users.
- **It has no idea what's technically expensive.** It may flag an edge case that
  costs a week and rate it the same as a one-line fix.
- **It doesn't know your org.** "No audit trail" is critical in a regulated company
  and noise in a five-person startup. It can't tell which you are unless you say.
- **Two runs will differ.** It's not deterministic. Re-running finds slightly
  different things — which is occasionally useful and always worth knowing.

**Use it as a checklist that argues back, not as an approval gate.** The verdict is
an opinion with reasoning attached; the reasoning is the valuable part.

---

## Why real user research is still required

FlowBreaker can tell you your document never says who sees a colleague's sick leave.

It cannot tell you whether your employees actually feel bad about their manager
seeing it, whether they'd stop using the system because of it, or whether they've
been quietly working around the current process for two years in a way nobody
documented.

It reads what you wrote. Not:

- Whether the problem you're solving is the problem your users have
- Whether your solution fits how they actually work
- Whether they'll use it
- Which of the twelve findings actually matters to them
- What they'd have told you if anyone had asked

In the worked example, FlowBreaker correctly identifies that "employees stop chasing
managers" is an unmeasured success criterion. It **cannot** tell you whether
employees find the chasing painful, or whether managers will decide any faster in a
tool than they did over WhatsApp. Nothing in the design changes the incentive — it
changes the interface. Only talking to those people settles that.

**A clean FlowBreaker report means your document is internally coherent. It does not
mean you're building the right thing.**

---

## Contributing

Contributions welcome — especially cases where FlowBreaker got it wrong.

The most useful thing you can file is a **miss** (a real defect it should have
caught) or a **false alarm** (a `critical` that wasn't). Both improve the question
bank in ways that guessing cannot. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache 2.0. See [LICENSE](LICENSE).
