# Dev Operating Model — Generic Core

A brand-free, portable operating model for running software projects with Claude across its two
surfaces: a coordination/reasoning surface (Claude.ai) and an agentic build surface (Claude Code).
This is the distilled, reusable core of a methodology refined in production and published so others
running similar projects can adopt it instead of starting from a blank page.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

## What's in here

- **`methodology/Portable-Operating-Model.md`** — the backbone. Separates *why* a practice exists
  (principle) from *how* one project did it (instantiation), so you adapt rather than cargo-cult.
  Start here.
- **`methodology/PI-core.md`** — Project Instructions (generic core): the always-on disciplines a
  coordinator and build team carry in-thread — the Story/Brief split, hard-stop rules, verification
  discipline, defense-in-depth, branching/PR workflow, orchestration-primitive selection.
- **`methodology/CLAUDE.md`** — the in-repo distillation of the PI: the lean, always-loaded
  operating contract for a Claude Code build session.

It names no project, person, or product by design. You add your own project layer (domain, stack,
invariants) on top.

## Who it's for

Solo operators and small teams running Claude-based software projects who want a tested methodology,
especially anyone coordinating work across Claude.ai and Claude Code.

## How to adopt

See **`ADOPTING.md`**. In short: read the Portable Operating Model, drop `PI-core.md` and
`CLAUDE.md` into your project, then layer your project-specific facts on top.

## Versioning

This is a **marked snapshot** — methodology as of the version stamped in `CHANGELOG.md`. It is
re-published on call when the underlying method moves, not continuously mirrored.

## License

Released under [CC BY 4.0](LICENSE): use, adapt, and build on it (even commercially); just credit
the source. Attribution: `<your-name-or-handle>` (replace before publishing).
