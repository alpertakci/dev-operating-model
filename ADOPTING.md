# Adopting this operating model

A short path to standing this methodology up on your own project. It assumes you run work across two
Claude surfaces — Claude.ai for coordination, Claude Code for the build. Read
`methodology/Portable-Operating-Model.md` first; it explains *why* each practice exists vs. *how*
one project implemented it, so you adapt the mechanism to your constraints. The steps below are the
short version of its adoption checklist.

## 1. Map the roles onto the surfaces
- Coordinator → Claude.ai. Decides and authors; cannot self-trigger, push, or touch a repo.
- Tech lead, coder, quality + security reviewers, quality engineer → Claude Code (holds repo/disk
  ground truth).

Accept the coordinator's limits and design around them; don't hand it execution.

## 2. Drop the core into your repo
- Copy `methodology/CLAUDE.md` to your repo root as `CLAUDE.md`.
- Put `methodology/PI-core.md` into your coordinator's standing project knowledge.
- Keep the Portable Operating Model as reference.

## 3. Add your project layer on top
The core is brand-free. On top of it write your project-specific facts: domain, stack, the
invariants your problem space demands, and any domain seats your team needs. Where your layer and
the core disagree on a project fact, your layer wins; where a project instruction would violate a
core framework constraint, that's a hard stop — cite it and route it.

## 4. Write your hard-stop list
Especially your irreversibility threshold for *your* destructive actions: prod deploy, customer-data
migration, key rotation. Gate them with an operator-shell confirmation the agent can't produce.

## 5. Wire verification to your real probes
CI output, deploy/runtime logs, health checks, DB state. The discipline is fixed (outputs verbatim;
confidence is not verification); the probes are yours. For UI work, stand up a browser-level verify.

## 6. Choose your enforcement layers
Deterministic checks (hooks, CI gates, branch protection) for safety properties; single checks for
design choices. Don't over-armor a design decision.

## 7. Set your handoff format
A durable record that lets a fresh session resume without re-deriving. Anchor on a high-water mark,
not a date.

## 8. Calibrate, then revisit
Match ceremony to demonstrated need — never default heavier "to be safe." Schedule a post-mortem to
simplify what didn't earn its keep.
