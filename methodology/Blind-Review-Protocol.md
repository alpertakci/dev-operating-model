<!-- BLIND-REVIEW PROTOCOL (PUBLIC EDITION). Brand-free. The default review mechanism when the
 tech lead declares PRIMITIVE: SUBAGENTS. Carries author≠verifier without team constitution: same
 model, same seat rubric, fresh context — identical verdict quality to a teammate reviewer; what
 teams natively give (an unmediated verdict channel) is bought back here by the artifact convention.
 Adapt the <SEAT> placeholders to your project's seat names. -->

# Blind-Review Protocol v1 (subagent pipeline)

## The three guards (all load-bearing; skipping any re-opens self-review)
1. **Blind, rubric-only spawn.** The reviewer subagent receives ONLY the fixed template below —
   never the author's claims, the build report, the lead's framing, or who wrote the code. The TL
   fills the `<>` slots and changes nothing else; a TL-improvised review prompt is the contamination
   vector this template exists to remove.
2. **Reviewer reads ground truth itself.** The diff from disk (`git show <hash>`), files, tests —
   never a summary handed down.
3. **Unlaundered verdict.** The reviewer writes its own artifact to `reviews/<hash>-<seat>.md`; the
   lead relays the path, never re-authors or summarizes the verdict into the close. *Scoped
   exception to reviewer read-only-by-design:* reviewers may use Bash to write their ONE verdict
   file under `reviews/` — and nothing else; `src/` and product code stay untouchable.

## Fixed spawn template (one subagent per seat; QA/SA always, QE when a UI/API surface is touched)
```
You are <SEAT> — load your definition at .claude/agents/<seat-file>.md and apply its rubric.
Review commit <HASH> on branch <BRANCH>. This prompt is your entire input: you have no author
identity, no build report, and no claims about the work — do not request or accept any.
Read the ground truth yourself: `git show <HASH>`, plus any file or test you need. Run what you
need to run (tests, greps, the app where your seat requires it).
Write your verdict to reviews/<HASH>-<seat>.md containing:
  VERDICT: PASS | BLOCK
  FINDINGS: each with file:line and severity (or "none")
  EVIDENCE: every command you ran with its verbatim output (V1; "(no output)" written explicitly)
Then report back exactly: "verdict written to reviews/<HASH>-<seat>.md" — nothing else.
```

## Lead handling rules
- Spawn QA/SA (and QE where due) as **parallel** subagent calls — the parallelism of a team, kept.
- On return, `cat` each verdict artifact and paste it **verbatim** into the PR / close report (V6:
  claims cross surfaces with outputs intact).
- Any BLOCK → route per the seat's escalation; the TL never overrides a BLOCK, only routes it.
- A verdict artifact missing at close = the review did not happen (V2) — no close.
- Where the project wires a task-close hook, it gates on the artifacts existing at HEAD for the
  commit under close (deterministic layer; PC re-points the existing hook to this convention).

## Honest limits (unchanged from teams — not a regression)
Defends against drift and self-preference, not deliberate lead fabrication across every gate — that
is caught by the operator's merge gate and PR read, exactly as with teams. Weights-level blind spots
are shared by any Claude reviewer, teammate or subagent; the human gate and hooks cover that layer.

---
*Blind-Review Protocol — public edition. Companion to PI-core.md §5/§7.*
