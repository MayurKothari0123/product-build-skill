# Contributing to FlowBreaker

Thanks for considering it. FlowBreaker is a prompt, not a program — which makes
contributing unusual, and unusually accessible. You don't need to write code. You
need to have used it on a real spec and noticed where it was wrong.

---

## The most valuable contribution: tell us what it got wrong

FlowBreaker's quality lives almost entirely in §5 (analysis areas) and §6 (question
bank). Those improve from real cases and essentially not at all from speculation.

Two kinds of report, both valuable:

### Misses — a real defect it should have caught

Open an issue titled **`Miss: <short description>`** with:

1. A minimal PRD excerpt reproducing it (anonymised — see below)
2. What FlowBreaker reported
3. The defect it should have caught
4. What surfaced it in the end (implementation, QA, production, a user)

Misses are the highest-value reports we get. Every one is a gap in the question bank
with a real consequence attached.

### False alarms — a finding that wasn't real

Open an issue titled **`False alarm: <short description>`** with the same structure,
plus **why** it didn't apply in your context.

These matter just as much. A tool that returns five criticals on every document
teaches people to skim past all five — including the one that mattered. Precision is
not a nice-to-have here; it's the difference between a tool that gets used and one
that gets ignored by week two.

**Anonymise before posting.** Reduce to the smallest excerpt that reproduces the
issue and replace real names, products and figures. A generic three-line excerpt is
more useful than a real fifty-line one anyway.

---

## Changing the skill

`FLOWBREAKER.md` is the entire product. Three constraints:

1. **It stays self-contained.** Someone who downloads only that file gets a fully
   working skill. No dependencies on sibling files.
2. **It stays under ~1,500 lines.** Past that, an agent starts skimming and §1 is
   the first casualty. Currently ~1,300. If your change pushes it over, propose
   what to cut, or open an issue about splitting reference material out.
3. **§1 wins.** The rules section is policy; everything else is procedure. If a
   change makes a later section conflict with §1, the later section is wrong.

### Adding to the question bank

Include the default risk level and a one-line justification. Ask yourself the same
question the skill asks itself (R6): **would the answer change product behaviour,
UX, implementation, security, or evaluation?** If not, it doesn't belong — the bank
is valuable because of what it excludes.

Say whether the area is Tier 1 (core, always runs) or Tier 2 (conditional), and for
Tier 2 give the trigger. Tier 1 additions need a high bar: every one lengthens every
review anyone ever runs.

### Changing artifact templates

If you change a template in §7 or the HTML in §9, **regenerate the affected
examples**. `examples/*/output/` are golden references, and stale ones make the repo
misrepresent its own output.

---

## Adding an example

Examples are documentation and regression tests at once. A good one has:

- **A realistic PRD** with *plausible* flaws — the mistakes real drafts make, not
  strawmen. The leave-request example has a stale section contradicting a current
  one, which is the most common real-world PRD defect and the least dramatic.
- **A domain the existing three don't cover.** Current: approval workflow, offline
  mobile, event ingestion.
- **`prd.md` plus at minimum `output/report.html`.**

### Adversarial fixtures

`examples/adversarial/` holds specs that test FlowBreaker's judgement rather than
its coverage. The most important is **a genuinely complete PRD it must NOT find
criticals in.** If you can write one it cries wolf on, that's a bug worth filing.

---

## Testing a change

No test runner — there's no code. Verify by hand:

1. **Install promise.** Copy *only* `FLOWBREAKER.md` into an empty directory with a
   PRD. Run a review. If it needs anything else from the repo, the change broke the
   core promise.
2. **GATE 1 holds.** With a PRD containing a genuine critical gap, it must stop
   after questions and not pre-generate flows.
3. **False-positive control.** Against a complete PRD, no manufactured criticals.
4. **Reference integrity.** Every `REQ-`/`FLOW-`/`Q-` referenced in the output
   exists.
5. **Report renders.** `report.html` opens offline, no console errors, no external
   requests, readable in both light and dark.
6. **Portability.** If you changed anything structural, run it in a second agent.
   Cross-agent portability is a stated goal.

---

## Style

Match what's there: direct, specific, no filler. The skill tells an agent what to do
and why, in that order. Where a rule exists because of a specific failure mode, say
what the failure mode is — an instruction with a reason attached gets followed more
reliably than one without.

---

## Code of conduct

Be decent. Assume good faith. Critique the spec, not the person who wrote it — which
is, appropriately, the same thing FlowBreaker is for.
