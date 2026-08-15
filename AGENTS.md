# AGENTS.md

Instructions for AI coding agents working in this repository.

This repo contains **FlowBreaker**, a requirements and UX stress-testing skill. The
entire skill is one file: [`SKILL.md`](SKILL.md).

## Working in this repo

- `SKILL.md` is the product. Everything else is documentation or examples.
- It must stay **self-contained**. Someone downloading only that file must get a
  fully working skill. Do not split content into sibling files it depends on.
- Keep it under ~1,500 lines. Past that, reference material should move into a
  `flowbreaker/` directory and the install story changes — a deliberate decision,
  not a drift.
- The non-negotiable rules live in §1 and are ordered first on purpose. Later
  sections are procedure; §1 is policy.
- `examples/*/output/` files and the root `report.html` are **golden references**. If
  you change a template in §7 or §9, regenerate the affected example or the repo
  starts lying about its own output format. `report.html` lives at the repo root
  rather than under `examples/*/output/` because it is the artifact people are sent
  a link to; the Markdown artifacts stay with their example.
- **R9 governs the report.** Each fact appears exactly once, in the issue it belongs
  to. If you add a section to §9, check it does not restate something an issue entry
  already carries — that is the failure the rule exists to prevent.

---

## The snippet to copy into YOUR project

Everything above is about developing FlowBreaker. To *use* it in your own repo, add
these lines to that repo's `AGENTS.md`:

```markdown
## Product & requirements review
When asked to review a PRD, feature spec, user flow, or acceptance criteria —
or when asked "what's missing here" about a product requirement —
read ./SKILL.md and follow it exactly.
Do not skip its clarification-question gate.
```

Then drop `SKILL.md` in that repo's root.

**Keep the pointer short; leave the skill in its own file.** `AGENTS.md` loads into
every conversation in a repository, so inlining the full skill costs tokens on every
unrelated turn and dilutes attention while someone is fixing a CSS bug. The pointer
costs three lines and loads the rest only when it applies.
