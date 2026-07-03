# Changelog

All notable changes to this public release are documented here. This repo is a marked snapshot —
each release stamps the methodology version it carries. Format based on Keep a Changelog.

## [1.1.0] - 2026-07-03

### Changed
- Updated to methodology as of generic core **v12** (from v9). The build primitive flipped: the
  **SUBAGENTS pipeline is now the default build loop** and **Agent Teams is the declared exception**.
  Also folded: **Rule 9 (agent-team constitution proof) finalized to its v2 form** — preflight →
  constitution-proof → recovery, with the spawn-mode correction (a named, backgrounded teammate vs.
  an anonymous one-shot subagent); effort inherited from the lead; model escalation as a tech-lead
  request; the §0.3 integrity-invariant bound; QE two-gate wording.
- `methodology/PI-core.md` (v12) and `methodology/CLAUDE.md` (v11) re-issued at the v12 snapshot.

### Added
- `methodology/Blind-Review-Protocol.md` — the default author-blind review mechanism for the
  SUBAGENTS pipeline (fixed rubric-only spawn; reviewer reads ground truth from disk; verdict as an
  artifact).
- `methodology/roles/` — five brand-free seat templates (TechLead, Coder, QualityReviewer,
  SecurityReviewer, QualityEngineer). Each is an agent definition a project copies to
  `.claude/agents/<px>-<seat>.md`, replacing the `<PX>/<px>` tokens and layering its own machinery
  below the marked PROJECT LAYER line.

## [1.0.0] - 2026-06-30

### Added
- Initial public release. Methodology as of generic core v9.
- `methodology/Portable-Operating-Model.md` — the portable backbone (principle vs. instantiation).
- `methodology/PI-core.md` — Project Instructions, generic core.
- `methodology/CLAUDE.md` — the in-repo distillation for a Claude Code build session.
- `ADOPTING.md`, `CONTRIBUTING.md`, and the CC BY 4.0 `LICENSE`.
