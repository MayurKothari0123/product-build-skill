# AGENTS.md

Instructions for AI coding agents working in this repository.

This repo contains **FlowBreaker**, a requirements and UX stress-testing skill. The
entire skill is one file: [`SKILL.md`](SKILL.md). Everything else is documentation or
examples. Installation and usage are in the [README](README.md) — don't restate them
here.

## Working in this repo

- `SKILL.md` must stay **self-contained**. Someone downloading only that file gets a
  fully working skill. Do not split content into sibling files it depends on.
- Keep it under ~1,500 lines. Past that, an agent starts skimming and §1 is the first
  casualty. Currently ~1,364. §9 (the HTML template) is the largest block that carries
  no instructions — trim there before anywhere else.
- The non-negotiable rules live in §1 and are ordered first on purpose. Later sections
  are procedure; §1 is policy. Rules defined elsewhere (R11 in §8, R12 in §3) carry a
  pointer in §1 so that list stays authoritative.
- **R11 governs the report.** Each fact appears exactly once, in the issue it belongs
  to. If you add a section to §9, check it does not restate what an issue entry already
  carries — one defect stated five ways is still one defect.
- **R12 governs the questions.** Sequence coupled questions, frame each with real
  options. If you add to the §6 bank, the addition needs plausible options, not just a
  gap to point at.
- **Phase A writes nothing** (§3). If you add a step that files something during the
  conversation, you have reintroduced the eleven-file problem. Four files plus the
  report; adding a fifth needs a reason that survives R11.
- `examples/*/output/` and the root `report.html` are **golden references**. Change a
  template in §7 or §9 and you must regenerate the affected example, or the repo starts
  lying about its own output format. `report.html` sits at the repo root because it is
  the artifact people are sent a link to; the Markdown artifacts stay with their
  example.
- Contributions are closed (see the README's policy). This file is for authorized
  collaborators and for the agents they run.
