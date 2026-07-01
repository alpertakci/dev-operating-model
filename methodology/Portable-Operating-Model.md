# A Portable Development Operating Model
### Methodology transfer for a new (customer-facing, likely cloud) software project

**Version 3** · supersedes v2 — if you already have v2, jump to **Net delta — v2 → v3** at the end;
the rest is unchanged.

This document hands a fresh project coordinator ("PC") a working development methodology,
stripped of the originating project's facts. It was refined on an internal, on-premise,
low-volume system of record run largely by one human director plus **Claude** agents running
across distinct Claude surfaces (Claude.ai and Claude Code — see §2). **Your project is
different** — customer-facing, probably cloud-based, probably a larger team — and those
differences should *change* some of the choices below, not just rename them. The document is
built to make that adaptation explicit.

You are staying on Claude. So this is not about choosing a model — it is about choosing **which
Claude surface runs which role**, and adapting the surrounding mechanism to your constraints.

This is also a **living document**: as the platform ships capabilities that sharpen the
methodology, they fold in here as a new version, each with a net delta at the end so a PC holding
an older copy can patch forward.

---

## 0 · How to read this (the one rule that governs all the others)

Everything below comes in two layers:

- **Principle** — *why* a practice exists. These transfer wholesale.
- **Instantiation** — *how* the source project did it, given its specific constraints. These are
  examples, not commandments. Replace them with the mechanism your constraints call for.

**The meta-rule: never adopt a mechanism whose justifying constraint you don't share.** The
source project couriered documents between two Claude surfaces by hand because its coordinator ran
on a surface that couldn't self-trigger or push, and its executor ran on an isolated on-prem box.
If your executor side can push and schedule (it can — see §2), the "up" half of that courier
evaporates — keep the *principle* (durable records, verified ground truth) and drop the
*workaround*. Copying a workaround whose cause you don't have is how methodology turns into cargo
cult.

Calibrate ceremony to demonstrated need, not to the sophistication available. A post-mortem that
can only ratify the thing it set out to build is not a post-mortem.

---

## 1 · What's different about your project (read this before anything else)

Four deltas reshape the choices downstream. Each is flagged inline later as **[ADAPT]**.

1. **Customer-facing** (vs. an internal tool with a known, small user set). Real users, real
   support load, possibly SLAs. The public attack surface is larger. Privacy/PII is likely in
   scope. Accessibility matters. **Operational monitoring of the deployed product becomes
   first-class** — not something you defer (see §10, the most important correction this document
   carries).

2. **Cloud-based** (vs. a single on-prem machine). You get CI/CD, multiple environments
   (dev/staging/prod), infrastructure-as-code, managed secrets, and real observability — and you
   *lose* the excuse that automation is "gated" or "disk-bound." Scheduled and automated jobs are
   first-class, not workarounds.

3. **Likely a larger / real team** (vs. single-author-equivalent). Branching weight, review
   throughput, seat allocation, and on-call all scale up. "A human confirms every deploy" stops
   being realistic; you need a pipeline.

4. **A real compliance posture.** The source project had *no* certification obligation and built
   its evidence trail "evidence-grade by intent" as a hedge. A customer-facing product touching
   customer data probably has **actual** obligations (GDPR, SOC 2, sector rules). Design the
   evidence trail to the real obligation, not as a learning exercise (see §9).

---

## 2 · The Claude surfaces and the agent topology (this is not generic "AI agents")

The role split exists *because the Claude surfaces have different reach*. The most common failure
in adopting this model is mis-routing a role onto a surface that physically can't do it — so name
the surfaces first, then map the roles onto them.

### The surfaces and their reach

| Surface | What it is | Reach — can / cannot | Role it runs |
|---|---|---|---|
| **Claude.ai (chat + Projects)** | The reasoning/strategy surface; holds standing project knowledge in a Project | Strong reasoning, document authorship, holds context. **Cannot** self-trigger on a schedule, **cannot** push notifications, **cannot** run code or touch a repo/disk. Reads connectors (Drive, etc.) **read-only**, only when a file is surfaced to it. | **Coordinator** |
| **Claude Code** | The agentic dev surface (CLI / IDE / desktop), operates on a real repo | Repo on disk; runs git, tests, builds; **holds ground truth**. Hosts **hooks** (deterministic enforcement at the moment of action), the **Agent Teams** substrate (coordinated multi-instance teams), **subagents** (in-session parallel workers), **Dynamic Workflows** (fanned background runs), and **Cloud Routines** (scheduled/event/API-triggered runs on Anthropic's cloud — machine-independent, connector-aware, can push). | **Tech lead, executor, quality + security reviewers** |
| **Claude Cowork** | The agentic knowledge-work surface (non-dev) | Claude Code's agency, but over documents / spreadsheets / slides instead of a code repo. | Outside the core dev loop — relevant only if your project has a heavy non-code ops/content workstream. |

### Two consequences to internalize before the roles

1. **The coordinator's limits are real, not stylistic.** A Claude.ai Projects thread genuinely
   cannot wake itself up, cannot push you a notification, and cannot read your repo or run a
   command. So the coordinator *decides and authors*; it never *executes*. Handing it execution
   duties is the canonical mis-route — and it is exactly the failure that, in the source project,
   once put the coordinator role on the wrong surface and broke work routing until corrected.
   **Your coordinator is the same surface as ours — Claude.ai chat/Projects — so these limits
   apply to you directly. Design around them; don't pretend they aren't there.**

2. **The enforcement layer lives on Claude Code, not Claude.ai.** Hooks, the audit/evidence trail,
   the deterministic gates (§7) are all produced on the Claude Code side, because that is the
   surface that runs at the moment of action and touches disk. The coordinator can *require* and
   *review* these (the read-time net), but it cannot *produce* them. Defense-in-depth (§7) is built
   on this split.

### The roles (each mapped to a surface)

- **Coordinator — Claude.ai (chat/Projects).** Owns the *what and why*: strategy, scope,
  rationale, domain knowledge, the ledger of work, and continuity across sessions. Decides and
  authors; does not implement (the surface can't).
- **Tech lead — Claude Code.** Owns the *how*: turns ratified scope into a concrete, executable
  build specification, and verifies the result against it. The sole interface to the executor.
- **Executor (coder) — Claude Code.** Implements, tests, commits on a branch.
- **Independent reviewers (quality + security) — Claude Code.** A quality backstop (correctness,
  coverage) and a security backstop (data flow, injection, authz, secrets). They review the
  executor's output, **never their own** — a reviewer that didn't write the code carries different
  biases and its own context, which makes the review more honest.

**The organizing split is Story vs. Brief**: the coordinator (Claude.ai) authors a *Story* (goal,
scope, rationale) that is ratified before the tech lead (Claude Code) authors a *Brief* (the
concrete build instructions), which is reviewed before the executor runs. Two gates, deliberately
staged across the two surfaces.

### The bridge — your #1 adapt point

Because the coordinator (Claude.ai) cannot reach the executor (Claude Code) directly, *something*
must relay between the surfaces: ratified scope down, verified progress up. The source project used
a **human courier plus a shared Drive folder**. You are on the same coordinator surface, so you
still need a bridge — but cloud gives you better materials than a hand-courier:

- The executor side (Claude Code) **can push and schedule** via Cloud Routines, so "up" traffic
  (progress, scheduled watches, alerts) can be automated rather than relayed by hand.
- The coordinator side (Claude.ai) still **can't self-trigger**, so "down" traffic (a new Story, a
  session-open) and the reconciliation ritual still need a trigger — a human, or an integration
  that drops the right artifact where the coordinator reads it.
- Keep the **principle** (a durable, verified relay of state across the surface boundary); pick the
  **mechanism** your cloud tooling affords. Don't reflexively copy "a human pastes between two
  windows."

### Seats and tier

- **Seat count is a hypothesis, not a truth.** The source used these roles for a one-director,
  low-volume system. A customer product with a real team may need more reviewer capacity, a
  dedicated release/on-call role, and a support-triage path. Don't pre-build seats — add one when a
  need is demonstrated, and be willing to remove one a post-mortem shows was idle.
- **Model tier, not model.** You are on Claude; the only dial is *tier/effort per role*. Use a
  capable tier for build *and* review — under-powering the correctness/security backstop to save
  tokens is the wrong trade when the stakes are real customers. Reserve the heaviest effort for
  genuinely hard work; never dial it up "to be safe."

---

## 3 · Review gates (who sits where)

**Principle.** Make every gate's owner explicit. Three gates:

- **Scope gate** — the coordinator (with the human) ratifies the Story before any Brief exists.
- **Implementation checkpoint** — the tech lead authors the Brief; a light check confirms it
  before the executor runs. The coordinator does *not* sit in this loop in real time; that's the
  point of the Story/Brief split — it lifts strategy off the per-change treadmill.
- **Deploy gate** — confirming a release is a deliberate, irreversibility-aware act.

**Name the faithfulness owner.** "Does the build actually implement the intent?" must be owned by
*someone* — if the coordinator is deliberately out of the per-Brief loop, then the operator's
check plus the reviewers carry it. Delegated-by-design is fine; *unowned* is a defect.

**Automated second-agent review before merge — a deterministic layer beneath the human gate.**
A reviewing agent that didn't write the code has its own context and different biases, so it
catches what the author's agent missed. On Claude Code this comes in tiers, manual to automated:
`/review` (a single-pass read of a PR in the terminal), the `/code-review` plugin (spins up
parallel subagents, each reading the diff from a different angle, scores findings by confidence,
and posts to the PR), and **Claude Code Review** (a managed service that runs on every PR via
GitHub). Wire one in *beneath* your human quality+security gate — always-on automated review,
human net on top (textbook §7 defense-in-depth). Two caveats: the managed service is
**Team/Enterprise-plan-gated**, and it **ingests your diff** — a data-flow decision if your code
carries anything sensitive.

**Bundle the mechanical workflow, not the judgment.** The build → verify → review → PR → CI-watch
sequence can be packaged as one callable scaffold (a skill that calls other skills) so the tech
lead doesn't re-author boilerplate per change. Bundle only the *mechanical* steps; the Brief's
substance stays the tech lead's judgment, and the scaffold must **never swallow the
irreversibility/merge gate (§5.7)** — the human confirmation stays a deliberate stop, not a step
the scaffold auto-runs.

**[ADAPT]** A cloud, customer-facing product turns the deploy gate into a **pipeline**, not a
single human merge: CI must be green, staging must pass, and risky releases go out behind a
canary/feature flag with a defined rollback trigger. Add an **operations/on-call gate** for
production incidents — a concern the internal source project didn't have.

---

## 4 · Change-size calibration

**Principle.** Size every change and scale the process to it:

| Size | Shape | Process |
|---|---|---|
| XS | config/doc/one-liner | executor → operator; no coordinator review |
| S | single-file logic, narrow refactor | executor → operator; coordinator review only if flagged |
| M | cross-file feature, new module | coordinator Story → tech-lead Brief → executor |
| L | schema change, new infra, architecture | add a coordinator architecture note before the Story |

**The sizing axis is blast radius and reversibility, not apparent complexity.** A scary-looking
refactor that's fully covered by tests and trivially revertible is smaller than a "one-line"
change to a live billing path. **Never default heavier "to be safe"** — that recreates the exact
ceremony the sizing exists to skip, and it trains everyone to stop trusting the sizing.

**[ADAPT]** Customer-facing raises real blast radius: a visible UI change, a migration on live
customer data, anything in an auth or payment path. Recalibrate thresholds *upward* where customer
impact is genuine — and only there.

---

## 5 · Hard-stop rules

**Principle.** Agents do not exercise judgment to paper over gaps. On any of the following, they
**stop, state the blocker, and route it** — no inference, no best guess, no partial deliverable:

1. A referenced input is missing.
2. Inputs contradict each other (state both; don't pick).
3. A decision is outside the agent's authority (state it and the correct routing).
4. A required input is absent.
5. A verification failed (state the discrepancy; don't proceed on the upstream claim).
6. An instruction violates a framework/platform constraint (cite it; route the conflict).
7. **Irreversibility threshold** — before anything hard to undo (secret/key generation, schema
   migration, data deletion, production deploy, account creation): stop and confirm. The threshold
   scales with how hard the action is to reverse, not how complex it looks.
8. **Cross-surface state divergence** — when the state an agent is working from (a handoff, a
   summary, a task entry, a claim made elsewhere) diverges from ground truth on its own surface:
   halt and route the divergence. The divergence *is* the next problem. (This rule matters *more*
   on a two-surface Claude setup — the Claude.ai coordinator and the Claude Code executor each see
   a different picture, and the gap between them is where errors hide.)

"Stop" means the response contains only the blocker and the routing. This transfers verbatim — it
is substrate-agnostic.

---

## 6 · Verification discipline (import this first; it is the crown jewel)

**Principle.** Distinguish *running* a verification from *claiming* one ran. They are
indistinguishable to the receiver unless the evidence is present.

- **Outputs verbatim.** A verification report carries the command and its literal output beneath
  it. "Verified, all green" with no output is rejected on sight. If a command produced nothing,
  say "(no output)" explicitly.
- **Confidence is not verification.** High confidence that a check *would* pass is never a
  substitute for running it. A more capable model lowers the base error rate; it does not replace
  the probe.
- **Claims survive boundaries.** A verification claim does not weaken by crossing a session,
  surface, or document boundary — it carries its outputs across intact, or it is rejected on
  arrival.
- **Scope expands after a violation.** When a claim is found output-less or fabricated, the next
  request explicitly probes whether the un-run check would have caught what it claimed to. Not
  punitive — it's how the system recovers without simply trusting the next claim.
- **Ground truth is the system, not the story about it.** Verify against the running service, the
  database, the actual file on disk, the CI log — never against a handoff, a summary, a plan, or
  an agent's recollection. **On this stack specifically: Claude Code holds disk/repo ground truth
  and runs the probes; the Claude.ai coordinator reasons over *surfaced excerpts* and must never
  assert disk state it hasn't had verified by the Code side.** Re-probe before acting on any
  "next step" a prior record describes.

This is the single highest-leverage thing to import, and it matters *more* for you than for the
source: a false "verified" here ships to paying customers.

**Encode every manual check you can as a feedback loop.** Claude already self-verifies against the
deterministic signals — type errors, lint, failing tests, runtime errors. What it *can't* infer
are the manual checks you run after it responds and the ones you run before you merge. Every such
step you encode (as a test, a gate, or a skill) pulls Claude's first response closer to the result
you actually wanted and lets it keep working without you turning the session into a turn-based
game. Write down the best-practice version of what you and your team already do, then encode it.
**But keep the discipline on top of the enthusiasm:** encoding a check and *trusting the agent's
claim that it ran* are different things. Encode aggressively; still surface the check's verbatim
output, never the self-report. More feedback loops and more capability lower the error rate — they
do not replace the probe (the confidence-is-not-verification rule above).

**[ADAPT]** The *discipline* is identical; the *probes* change with your stack — CI test output,
deploy and runtime logs, health-check endpoints, database state, synthetic monitors. Name them as
Claude-native skills where you can. For UI work (which a customer-facing product has and the
internal source did not), a **`frontend-verify` skill** is the canonical example: open the change
in a real browser (the Claude Code desktop embedded preview, or the **Chrome DevTools MCP** /
Agent browser from the CLI), check the console for errors, click through as the user would, then
run a performance trace and audit Core Web Vitals — fix and re-verify before responding. For a
customer-facing UI this is gospel, not optional. Author such skills by having Claude **interview
you** (the `skill-creator` plugin) rather than writing them cold; if you can't put the process into
words, ask Claude for the domain best-practices first.

---

## 7 · Defense-in-depth (and its limits)

**Principle.** For a property that must be true for safety or compliance — a verification claim
has its outputs, an irreversible action is gated, a secret never lands in a log — enforce it at
**more than one layer**:

- **Deterministic / automated layer** — on this stack, **Claude Code hooks**: they run at the
  moment of action and produce the evidence record. (In cloud terms: CI gates, pre-commit, branch
  protection, admission controllers, policy-as-code — and the automated pre-merge code review of
  §3 sits here too.)
- **Persistent-record layer** — the property is restated in whatever artifact crosses a boundary,
  so it survives the boundary.
- **Human/review layer** — catches what the automated matcher missed (the edge case, the novel
  pattern, the bug in the check itself). The Claude.ai coordinator is well-placed for this net,
  precisely because it can't produce the deterministic layer and must read the evidence to trust
  it.

No single layer is trusted alone for a safety property. **But single-layer is correct for a pure
design choice** (which library, which file layout, a naming convention). Three layers on a design
decision is over-armoring — the same ceremony inflation the size calibration exists to avoid.
Match enforcement weight to whether the property is safety-critical.

**Companion rule: agents for judgment, automation for rules.** Anything checkable by a
deterministic rule (file present, test green, string matches, claim has outputs) belongs in a hook
or CI gate, not an agent. Reserve agents for judgment that can't be reduced to a rule. This shrinks
the silent-failure surface and makes the evidence trail deterministic.

**[ADAPT]** Cloud gives you far more and better enforcement points than a single box did. Use them
as the deterministic layer — they're stronger and more durable than ad-hoc scripts — while keeping
Claude Code hooks for the agent-loop properties they're uniquely placed to enforce.

---

## 8 · Handoffs & session continuity

**Principle.** A handoff is a durable record that lets a fresh session resume **without
re-deriving anything**. It is not punctuation between turns.

- **Session-open ritual.** At the start of a working session, reconcile the standing inputs
  (dependency/doc watch, security findings, board state) against a **baseline anchor** — a precise
  pointer to the exact artifact you last reconciled (identifier + version high-water mark + the
  values you checked), *not* a date. You diff against a known point, not against "yesterday."
  Carry a guard for rolling-window sources (if the window has slid past your last anchor, fetch the
  prior snapshot and enumerate what aged off before concluding "no change").
- **Recall zones.** Put must-not-miss items (open decisions, next action, blockers) at the **top**,
  detail in the **middle**, the durable summary (decisions, deltas, watch sections, required
  actions) at the **bottom**. Models recall the start and end of a long document better than the
  middle — don't bury a blocker there.
- **Standing watch sections always present**, even when empty (explicit "(none)"). Absence-by-
  omission is how a standing reminder silently dies.
- **Drafted artifacts live inline, not by reference.** "Drafted in the prior session, ready to drop
  in" is a dangling pointer the next session can't follow. Reproduce code/prompts/templates in full.
- **A handoff is a prior-turn record, not current truth.** Re-probe ground truth before acting on
  any "next step" it describes; a divergence is a hard-stop (§5.8).

**[ADAPT]** Much of the source's handoff *ritual* existed because its coordinator (Claude.ai)
couldn't self-trigger and a human relayed state across the Claude.ai↔Claude Code bridge (§2). Your
coordinator shares that limit, so the session-open *trigger* still needs a human or an integration
— but the executor side can now feed the "up" channel automatically (Cloud Routines writing a
status artifact the coordinator reads at open). Keep the **durable-record** and **baseline-anchor**
principles intact; replace "human pastes between windows" with your real integrations.

---

## 9 · The evidence trail

**Principle.** Treat your audit/event log as the **evidence object** — the timestamped,
tamper-evident record of what happened. Code and docs merely *explain* how that evidence is
produced. Build it to evidence grade from day one, because it is cheap up front and expensive to
retrofit:

- Append-only; tamper-evident (e.g. hash-chained, so altering one entry breaks every later hash).
- Written only by the enforcement layer (the Claude Code hooks / your CI) — no direct agent or
  operator writes.
- Access-controlled; data-minimized (reference a path, not sensitive contents; redact before
  write); residency-aware; retained to a stated policy.
- Assert an **invariant** (the chain links end-to-end), never a brittle fixed count.

**[ADAPT — this is where your project diverges hardest.** The source had *no* certification
obligation and built this as a hedge and a learning exercise. A customer-facing product handling
customer data likely has a **real** obligation (GDPR records-of-processing, SOC 2, sector rules).
Design the trail to the actual requirement, map controls to the framework explicitly, and treat it
as a shipping concern, not a "nice to have." Conversely, don't gold-plate beyond your real
obligation.]

---

## 10 · Orchestration, automation — and the one category error to avoid

**Principle.** Match the orchestration topology to demonstrated need. The Claude Code primitives
each solve a *different* structural problem:

- **A single strong session** — most work.
- **Subagents** — in-session parallel workers for embarrassingly-parallel side reads; the parent
  only needs the summary.
- **Agent Teams** — a coordinated standing team (lead + teammates, each its own context) where work
  needs ongoing cross-check; the model used for the build loop here.
- **Dynamic Workflows** — a fanned background run for genuinely repository-scale, convergence-driven
  jobs; expensive, plan-gated; reserve for terminal/large passes.
- **Cloud Routines** — a scheduled / event / API-triggered run on Anthropic's cloud;
  machine-independent and can push — the right tool for recurring watches and for the automated
  "up" channel across the bridge.

Using the wrong one wastes tokens, time, or trust. Cost honesty: a coordinated team costs
materially more than a single session — justify it by the correctness gain at your stakes, and be
honest when the gain isn't there.

**The category error to avoid (the most transferable warning here):** **the channels you build to
coordinate the build are not the channels that monitor the deployed product.** The source project
conflated the two — it tried to repurpose its *build-time* coordination notifications (the
Claude.ai↔Claude Code bridge plumbing) to solve *production* monitoring, and even designed an alert
for a failure mode (the host being down) that real production signals already covered far better.
Keep them strictly separate:

- **Build-comms** — how your Claude agents/coordinator stay in sync *during development*.
  Scaffolding. Scoped to the build. (Cloud Routines + push live on the Claude Code side.)
- **Operational monitoring** — how you learn the *deployed product* is unhealthy. Permanent.
  First-class. Designed on its own terms, on your production stack — not on the Claude build loop.

**[ADAPT — for you this distinction is sharper, not softer.** Because you are customer-facing and
cloud-based, operational monitoring is genuinely first-class: uptime, error rates, latency,
alerting, on-call, incident response, SLOs. The source project could *defer* production alerting
because its "users" were a handful of internal staff who'd just email when something broke — you
cannot. Design operational monitoring deliberately and early, and never let build-time coordination
plumbing masquerade as it.]

---

## 11 · The meta-discipline

Two habits keep the whole thing from ossifying:

- **Calibrate to demonstrated need.** Match ceremony, topology, and enforcement weight to evidence
  of need — not to the sophistication that happens to be available. The most elaborate multi-agent
  pipeline often loses to a single well-prompted Claude session; be willing to find that out.
- **Run honest post-mortems.** Periodically evaluate the model against evidence — token cost
  actuals vs. assumed, operator-load actuals, whether each seat earned its keep — and be willing to
  *simplify or re-scope*, not just ratify. The same calibration that governs ceremony governs the
  methodology itself.

---

## 12 · Adoption checklist for the new PC

1. **Map the four concerns onto the Claude surfaces (§2):** coordinator on **Claude.ai
   chat/Projects**; tech-lead, executor, and quality+security reviewers on **Claude Code**. Confirm
   you accept the Claude.ai coordinator's limits (no self-trigger, no push, no disk/code, read-only
   connectors) and design around them.
2. **Define your bridge (§2).** Decide how scope goes down and verified progress comes up across
   the Claude.ai↔Claude Code boundary — automate the "up" channel with Cloud Routines/push; pick a
   trigger for the "down" channel.
3. **Pick tiers** per role; don't under-power review.
4. Write your **hard-stop list** (§5) — especially your irreversibility threshold for *your*
   destructive actions (prod deploy, customer-data migration, key rotation).
5. **Wire the verification discipline (§6)** to your stack's real probes (CI, logs, health checks,
   DB) — with Claude Code as ground truth and the coordinator reasoning over surfaced excerpts.
   Encode your manual checks as feedback-loop skills (e.g. `frontend-verify`); surface their output.
6. **Choose enforcement layers (§7):** Claude Code hooks + CI gates / branch protection /
   policy-as-code for safety properties; single checks for design choices. Add an automated
   pre-merge code review (§3) beneath the human gate.
7. Define your **handoff format and baseline anchors** (§8).
8. Design your **evidence trail** to your **real** compliance obligation (§9).
9. Stand up **operational monitoring** as its own first-class system, separate from build-comms
   (§10).
10. Pick a starting **orchestration topology** (§10) and schedule a **post-mortem** to revisit it
    (§11).
11. Size your first few changes (§4) and resist defaulting heavier "to be safe."

---

## Net delta — v2 → v3 (for a PC who already has v2)

Everything new in v3 is on the **Claude Code / executor side**. None of it changes the surface
model, the bridge, the coordinator's role, or the principles — it adds Claude-native tooling to
*instantiate* checks the methodology already required, drawn from Anthropic's "feedback loops"
guidance (which validates the core: self-verification loops while building + an independent reviewer
that didn't write the code = our Verification Discipline + the QA/SA gate).

- **§3 — automated pre-merge review added.** A deterministic second-agent review tier *beneath* the
  human quality+security gate: `/review` (terminal single-pass), `/code-review` (parallel subagents,
  confidence-scored, posts to the PR), and Claude Code Review (managed, runs on every PR). Caveats:
  the managed service is **Team/Enterprise-plan-gated** and **ingests the diff** (data-flow
  decision). Plus a note to bundle the mechanical build→verify→review→PR→CI workflow as one callable
  scaffold — without ever letting it swallow the §5.7 merge gate.
- **§6 — feedback-loop principle + named UI probes added.** Encode every manual check you can as a
  deterministic feedback loop so Claude's first response lands closer to target and runs with less
  babysitting — *while keeping the rule that the check's output is surfaced, not the agent's claim
  it ran.* Named a concrete **`frontend-verify`** skill (browser → console → click-through → Core
  Web Vitals via **Chrome DevTools MCP** / Agent browser) and the **`skill-creator`** interview
  pattern for authoring skills.
- **§7 — one cross-reference.** The deterministic layer now notes that the automated pre-merge
  review (§3) sits there. No change to the principle.
- **Unchanged in substance:** §0–§2 (surfaces, bridge, topology), §4, §5, §8–§12.

**How to patch forward without re-reading:** add the automated-review tier to your deploy/review
gate, add "encode manual checks as surfaced-output feedback loops" to your verification setup, and
keep `frontend-verify` + Chrome DevTools MCP in your back pocket for when UI work starts. Nothing
already adopted from v2 needs to change.

---

*This is a methodology, not a configuration. You're staying on Claude — so adopt the principles,
keep the surface mapping that the role split depends on, and rebuild the bridge and the
enforcement plumbing for your own cloud constraints. Where this document and your reality disagree,
your reality wins — and that disagreement is the most useful thing in here.*
