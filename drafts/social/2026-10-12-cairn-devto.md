---
title: "Cairn: A Work Harness for AI Coding Agents — Plans in Git, Truth in Your Tracker, Nothing Done Without Evidence"
published: false
description: "What cairn is, what it does, the full feature list, and the technical nitty-gritty of how an MCP server plus a few markdown verbs turns an AI coding agent into something you can hand a real project — and audit afterward."
tags: ai, mcp, devtools, productivity
canonical_url: https://www.eventually-consistent.io/projects/cairn/
cover_image: https://www.eventually-consistent.io/img/cairn-logo-h.jpg
---

<!-- dev.to draft. Posting is John's click. Tags max 4 on dev.to; swap `productivity` for `opensource` or `claude` if preferred. -->

Ever asked an AI coding agent to build something *real* — not a demo, a
project with phases and a deadline and a boss who asks how it's going —
and watched the plan evaporate the moment the context window rolled over?

I have. Repeatedly. The agent was brilliant for an afternoon and amnesiac
by morning. The plan lived in a chat scrollback. Lessons learned lived
nowhere. And the issue tracker my team actually looked at never heard
about any of it. Worse: when I asked the agent whether something was done,
it said yes, because closing the ticket was easier than testing it.

Cairn is what I built to fix that, and this is the full write-up: what
it's for, what it does, every feature worth naming, and then the technical
nitty-gritty of *how* — because the how is where the honesty lives. It's
open source (MIT), it's on npm, and it has been dogfooding itself through
25 phases, 325 issues, and eight releases since August…

## What it is, in one paragraph

Cairn is a **work harness for AI coding agents**. It ships as a Claude Code
plugin — 39 `/cairn:<verb>` slash commands backed by a TypeScript MCP
server exposing 87 typed tools — and a one-command installer wires the
same server and verbs into seven other harnesses (Grok Build, Copilot CLI,
Codex, Gemini CLI, Cursor, OpenCode, Zed). The agent plans in phases,
mirrors every phase and requirement into the team's real issue tracker,
claims and closes issues as it works, verifies phases *goal-backward*
against what they promised, and refuses to ship when the plan and the
tracker disagree. Alongside the lifecycle: durable memory with provenance,
crash-proof session continuity, cross-vendor adversarial review, budgeted
headless execution, docs publishing, a portfolio board, and guards that
refuse destructive or self-modifying actions in one plain line.

The name is the point. A cairn is a stack of stones that marks the trail
for whoever comes next — including future you, who has forgotten
everything.

## The design law: tracker-first

Every other decision in cairn descends from one rule: **the issue tracker
your team already reads is the single source of truth for work.** Not a
`.json` in the repo. Not the agent's context. The tracker.

Why so blunt? Because the tracker is the one place a non-engineer will
look. Your PM, your manager, your future self three months from now — they
open GitHub Issues or Jira or Linear, and whatever is there *is* the
project. If the agent's plan lives anywhere else, you've built a second
reality that drifts from the first the moment nobody's watching.

So cairn splits state into three stores with three jobs:

| Store | Holds | Survives |
|---|---|---|
| Your issue tracker | work items — phases, issues, comments, worklogs | everything; it's the truth |
| Your git repo | prose — plans, decisions, memory cards, verification records | everything; it's versioned |
| `~/.cairn/` | caches — handoff files, the memory search index, metrics | nothing; losing it costs zero |

Eight tracker adapters sit behind one interface: GitHub, GitLab, Jira,
Asana, Azure Boards, ClickUp, Linear, and a zero-credential **local**
backend that stores issues as plain files in your repo with a real
dependency graph. Start local with no accounts at all; when the team
grows, one `tracker_migrate` command promotes the whole store to a hosted
backend with history, comments, links, and estimates carried over.

## The lifecycle in five commands

```
/cairn:new       interview → PROJECT.md, roadmap, phases → mirrored to your tracker
/cairn:plan 2    research the phase, write the task breakdown, reconcile drift
/cairn:work 2    claim issues, do the work, close on verified done
/cairn:verify 2  goal-backward check: does the code deliver what the phase promised?
/cairn:ship      refuses unless drift-clean and every verified phase is closed out
```

`new` interviews you — what you're building, what done looks like, three
to ten requirements, how the phases break down — and leaves two things
behind. In git: `PROJECT.md`, a `roadmap.md` phase table, and a directory
per phase with `CONTEXT.md` (locked decisions) and `PLAN.md` (tasks). In
your tracker: a phase (epic, milestone, or list — whatever your backend
calls it) per roadmap row and a real issue per requirement. Your manager
can open the board right now and see the whole project.

`plan` has a depth dial (`--quick`, standard, `--deep`), can ingest a PRD,
can run `--challenge` to have a plan-stage interrogation seat ask *who is
this for, what breaks, why now*, and must include an "approaches
considered" block or `plan_check` flags it.

`work` claims issues with a "Starting now" comment and the base commit,
posts progress at real milestones, and closes with the commit range and
typed test evidence. `--wave` fans independent tasks out to parallel
agents in isolated worktrees.

`verify` is the gate most tools skip. It doesn't ask whether the tasks
closed. It asks whether the codebase delivers what the phase promised,
writes `VERIFICATION.md`, and a closed-but-unverified issue becomes a drift
flag, not a success.

`ship` refuses unless drift is clean and every verified phase's issues are
closed. Since v2.8.1 the drift half is enforced by a hook, not a procedure
— more on that below.

## The feature list

Grouped by what you're actually trying to do. Every item here is shipped
and tested at v2.8.3.

**Planning and research**
- Phase planning with a depth dial, PRD ingest, TDD and MVP modes, gap analysis
- `survey` — project-wide research with discussed roadmap changes
- `scout` — research-only passes with resumable checkpoints across phases
- `route` — insert, remove, or edit phases; decimal inserts (N.1–N.9), never renumber
- `probe` — risk-ordered throwaway spikes with recorded verdicts
- `draft` — multi-variant design mockups on a shared theme, with a designer agent
- `import` — reverse-mirror an existing tracker epic into plan artifacts
- `map` — a queryable project knowledge graph (multi-hop queries, freshness metadata) that plan, scout, and survey read when it exists

**Tracker mirroring**
- Eight backends, one SPI, write-through with cached reads
- Claim / progress / close comments on every issue; measured durations; worklogs where the backend supports them
- Custom status vocabularies — your board's real state names with a semantic category underneath
- Sprint placement, story points, and time estimates on Jira scrum boards
- Image attachments on issues (native on Jira, files-in-repo locally)
- Agent cost per issue and per phase, approximate and labeled as such
- Inbound delta: `plan_tracker_delta` + `resync` ingest edits made outside the session
- Native Claude Code Tasks mirror into your tracker via hooks — no polling
- `triage` — open-issue sweep against project conventions; `--apply` for the safe subset

**Verification and review**
- Goal-backward `verify` with committed `VERIFICATION.md`
- Plan ↔ tracker drift math (six reasons: two errors, one staleness, two advisory, one repair)
- Five-axis `review` with data-defined seats and signal-based dispatch
- Refutation panel on every serious finding — CONFIRMED / PLAUSIBLE / REFUTED, tallied in code
- `audit` modes: uat, milestone, security (with a surface check), ui, eval, validation, tests, plans, docs, simplify, memory — and `sweep`, which runs them all and diffs against the last sweep
- `--fix` on audits and reviews: patches generated in a detached scratch worktree, judged by an independent verifier, applied only on your say-so
- `peers` — external AI CLIs as reviewers; `council` mode judges the product itself
- An append-only `LEDGER.md` per phase tying issues to commits, refusing lines without evidence or an explicit waiver

**Memory**
- `remember` / `recall` — git-committed cards with provenance, served `STALE` when the source moved
- Passive observation log → `retro` promotes real patterns to cards with confidence grades
- Role-scoped memory so review seats remember what they've seen
- `distill` — ship-time synthesis of plans and cards into public-safe `docs/`
- `brief` — onboarding briefing from cards and plans
- Retro-gated compaction; a memory self-audit that reads files and git, never the index

**Sessions and continuity**
- Crash-proof handoff refreshed by every state-changing tool; resume at the exact task
- `trace` — persistent debugging sessions (evidence → hypothesis → test) mirrored to the tracker
- `thread` — long-running context that survives `/clear`
- `waypoint` — explicit pause and resume
- `backtrack` — safe git undo by phase or plan manifest, reverts only
- `medic` — planning-directory health, repair, and workflow forensics

**Autonomy and operations**
- `auto --batch` — headless execution of chosen phases inside a token or dollar budget, with staged push pre-authorization
- `outlook` — portfolio board across every cairn project on the machine, staleness-flagged, cost rolled up
- `basecamp` — multi-project workspaces with a dispatch board
- `docs publish` — README and `docs/` to Confluence or Docusaurus, folder per project, idempotent
- `docs_drift` — deterministic detection of phases the changelog and docs haven't caught up with, with generation wired into verify / ship / summit
- `summit` — milestone completion: verify gate, tracker close, archive, tag
- `fast` — trivial inline change, one issue, ≤3 files, one atomic commit
- `mark` — zero-friction capture of backlog items or notes
- `do` — plain-English router to the right verb
- `profile` and `tune` — how cairn talks to you, and `cairn.json` configuration
- Three PreToolUse guards — leak, run, harness — plus a loop adviser

That's a long list, and the natural skeptical question is whether it's a
pile of features or a system. The rest of this article is my answer.

## The technical nitty-gritty

### The layer split, and why tests enforce it

The plugin layer — commands, skills, hooks — owns **policy and judgment**.
The server owns **every mechanism with a wrong answer**: state
transitions, tracker mirroring, drift math, staleness checks, duration
measurement, budget enforcement. If a thing can be computed, it is
computed, in TypeScript, with a test. If it needs judgment, it's a
markdown subroutine the agent reads.

```
plugin (thin)     commands + skills + hooks — policy and judgment
      │ typed MCP tools
cairn-server      tracker/ · planning/ · memory/ · core/   (one process)
      │
tracker API (work truth) · git repo (prose truth) · ~/.cairn/ (disposable cache)
```

The split isn't a diagram; it's asserted by `server/test/architecture.test.ts`
on every run. Four rules: tracker adapters are leaves; the tracker
subsystem imports none of its callers; docs connectors may reuse exactly
four named tracker modules and an unused exception fails the test;
`index.ts` is the composition root and the import tree has no cycles. And
every rule fails if its file pattern matches zero files — because a check
that cannot fail is worse than no check (that's ADR 0019, and it came from
finding timing tests that were measuring process startup: a bare spawn was
31 ms and the hook under test was 32).

### Verbs are markdown; tools are typed

Each of the 39 verbs is a markdown subroutine under
`skills/cairn-trailhead/verbs/`. The slash-command shims are generated
from a routing table, and CI fails if the table, the shims, and the verb
files disagree. Verbs sequence tool calls; they don't restate policy —
policy lives in two skills (`cairn-planning`, `cairn-memory`) every verb
inherits.

This is also why the multi-harness story is cheap. The server speaks plain
MCP over stdio. A "port" is: point the harness's MCP config at the server,
install the generated `AGENTS.md` verb registry, add a slash surface if
the harness has one. Cursor additionally gets a real port of the hook
layer — one adapter translates Cursor's hook events into the shape cairn's
scripts already speak, so continuity, the leak guard, observation capture,
and both PreToolUse guards run there unchanged.

The 87 tools fail with **typed error codes** — `AUTH_MISSING`,
`RATE_LIMITED`, `NOT_FOUND`, `TRACKER_REJECTED` (4xx, with next actions
derived from the body), `TRACKER_DOWN` (5xx), `CONFIG_INVALID` — and the
shared rule every verb inherits is: one plain-language line plus the
user's next action *first*, the code second, never a raw payload as the
lead. One backend being down never blocks git-side work.

### The tracker SPI

Eight adapters, one `TrackerAdapter` interface with per-backend
**capability flags**. The rule is *declare, don't flatten*: if Jira has
sprints and GitHub doesn't, the adapter says so and the caller adapts,
rather than everyone pretending to the lowest common denominator. Shared
HTTP core with retry and backoff. Write-through with cached reads. One
contract test suite every adapter must pass — fixtures in CI, live
sandboxes before release (29 hand-run drills live outside the unit suite
for that).

Config is two-level: a top-level block names the backend, and the
adapter's own Zod schema deep-validates the opaque config. Credentials
never enter the file — config carries only the *names* of environment
variables, and if a key or value looks like a secret, the loader throws.
Your `cairn.json` is safe to commit.

```json
{
  "tracker": {
    "type": "github",
    "config": { "repo": "owner/name" }
  }
}
```

The docs connectors (Confluence, Docusaurus) mirror the same pattern as a
sibling subsystem — their own SPI, capability flags, registry — reusing
only the HTTP core. Markdown crosses the SPI and each adapter converts
natively; unknown constructs degrade to escaped text, never fail. Page
identity is title plus ancestry, so re-publishing is find-then-update and
never duplicates.

### Drift is arithmetic

`plan_drift` scans every phase's `PLAN.md` frontmatter — which lists the
tracker issue ids that phase owns — and asks the tracker about each one.
Six reasons can fire:

- **missing** — the issue id no longer exists (error)
- **closed-unverified** — closed, and the phase has no `VERIFICATION.md` (error)
- **stale security audit** — the audit record no longer describes the commit it judged (staleness; stops ship and verify)
- **quiet in-progress** — claimed, no activity for a while (advisory)
- **aged branch** — a phase branch that hasn't moved (advisory)
- **roadmap repair** — the roadmap says "planned" but `VERIFICATION.md` exists; patch the cell

Advisory means it neither stops a ship nor fails a verification. Errors
block. Since v2.8.1, `plan_drift` **stamps the commit it checked**, and
the run guard hook refuses an agent push to the default branch when that
stamp is missing, dirty, or for a different commit. The ship gate used to
be a procedure the verb followed; now it's a mechanism the hook enforces,
with a named owner-only override for the rare legitimate case.

### Durations are measured, never asked for

This one came from an audit. The agent's self-reported time spent was
checked against reality across eight consecutive closes and ran six to
fourteen times over; one claimed 128 minutes against 10 measured. So ADR
0020: **the server measures.** A four-rung ladder, and the rung travels
with the number:

1. Claim-comment timestamp → close (the "Starting now" comment is matched strictly)
2. Server-observed in-progress transition → close
3. First-to-last commit span in the close evidence
4. None — stated as none, with a degraded marker

The agent's own claim is kept beside the measurement, labeled as a claim,
never substituted. The close leaves an ephemeral receipt keyed by issue
id; `ledger_append` consumes it into the ledger line (ADR 0021). Jira
worklogs receive the measured minutes. Your velocity chart stops being
fiction.

### Memory with provenance

Two tiers. Tier 1 is a disposable SQLite FTS5 index outside the repo —
fast recall, zero trust. Tier 2 is git-committed **cards**: one fact each,
with provenance — which file, which commit, which decision produced it.
On recall, the server diffs the cited source against the recorded commit
and serves the card flagged `STALE` if it moved. The rule is blunt:
*memory can be wrong; it can never silently lie.*

Cards accrete from work, not from vibes. A passive observation hook notes
what each tool call touched and whether it failed. At `retro`, real
patterns — churn hotspots, repeated error-fix loops — are promoted to
cards with a confidence grade that gets re-checked later; confirmed
lessons grade up, contradicted ones grade down. Compaction is retro-gated
rotation with no time decay (an archive cannot itself be archived), and
the memory self-audit reads files and git rather than the index, so a
broken native binding can't lie to it.

A small war story from v2.8.3, because it's exactly the kind of bug this
design is meant to catch: the observation buffer's "did this tool call
error" flag turned out to be structurally false. Claude Code emits no
PostToolUse event for a failed call — the failure arrives as a separate
PostToolUseFailure event, and nothing was listening. Any lesson card
citing error churn would have been fabricated. The fix was hook-side, and
the test now pins both the failure path and the distinction between an
error and a user interrupt.

### Continuity: who outranks whom

Every state-changing tool refreshes a handoff file write-through; hooks
cover the gaps (a throttled PostToolUse write, an unthrottled write before
compaction, injection at session start). Kill the session mid-task —
compaction, usage cap, `/clear`, a crash — and the next one resumes at the
exact task with zero re-executed work.

The part that makes it trustworthy is the **trust order**: the tracker
and `git log` outrank `LEDGER.md`, which outranks the handoff. A
checkpoint can never win an argument with reality. Handoffs older than 14
days are stale and never auto-resumed; a skeleton guard keeps handoff
richness monotonic; hooks target under 100 ms.

### Review seats, and the refutation panel

A review **seat** is data, not a prompt: one Zod schema with a name, a
one-line lens, rubric categories, an anchored 0–10 scale with a
fabrication-refusal honesty line, a *required* injection dose
(minimal / standard / full), a convening stage (review / plan / any),
scope signals, and an advisory model preference. Six ship by default —
correctness, clarity, architecture, security, tests, and a plan-stage
interrogation seat — and the first five reproduce review's classic axes
byte-identically, regression-pinned. Projects override in
`.cairn/roles/*.md`.

Signals in the diff pick which seats convene (blast radius always wins;
security never gets pruned). However many seats raise a finding, **one
finding set** comes out: dedup by file, two-line window, and claim
overlap before severity ranking, crediting every raiser.

Then the **refutation panel**. Every finding carries a typed
`failure_scenario`. Critical and important findings face independent
votes of CONFIRMED / PLAUSIBLE / REFUTED, tallied in code: a refuted
majority kills the finding (it stays in the record with its votes, never
filed), ties survive as plausible, and a critical finding on a security
scope with fewer than two votes cannot be recorded at all. Audit records
are stamped with the commit judged and a dirty flag, and a stale security
audit is a drift flag that stops ship and verify.

`--fix` is deliberately slow: a dirty tree is refused, fixes are generated
in a detached scratch worktree, written as patch files mirrored to their
issues, judged by one independent verifier stating three claims (targeted,
no new issue, behavior unchanged), and applied only on your choice.

### `audit sweep`, and what "the same finding" means

`sweep` runs every audit leg in order — security, the security surface
check, tests, docs, plans, simplify, memory, milestone view last. Duplicates
collapse, the refutation panel judges every serious finding, and only
survivors reach the tracker, labeled with the leg that found them. One
dated manifest per sweep classifies each finding against the previous
sweep as **new / persisting / fixed / regressed**, and a reworded finding
is still the same finding. The first sweep says "first baseline."

The **security surface check** is the leg I'm proudest of and least
comfortable with. It maps every imperative rule in the instructions,
procedures, and hook prose to the deterministic control that enforces it.
Run on cairn itself, it found 58 of 258 rules with nothing behind them.
That number became the v2.7 and v2.8 roadmap.

### The three guards

All three are PreToolUse hooks — dependency-free Node scripts that read
the tool call before it runs and exit 2 to refuse. They share one shell
reader (`shell-words.mjs`) that understands keywords, grouping, case-folded
names, wrapper flags, program runners (`xcrun`, `arch`, `strace`,
`runuser`, `pkexec`), `cd`, `git -C`, `--git-dir`, `GIT_DIR`, `xargs`,
`find -exec`, `su -c`, archives, patches, downloads, and `$(…)` /
backtick / `$((…))` / process substitutions nested up to eight levels.
A deeply nested line that used to run the guard for seconds takes about
90 ms.

- **Leak guard** blocks a `git commit` whose *staged* diff would carry
  planning-directory paths, phase refs, cairn labels, or tracker ids into
  source files. It detects widened commits (`-a`, `--all`, pathspecs) and
  scans accordingly. The same pattern module scrubs `distill` and `ship`
  output and leak-scans everything `peer_run` sends to an external CLI.
- **Run guard** refuses a branch checkout, branch switch, or hard reset
  aimed at the directory you're typing in while a headless run owns a
  worktree; refuses `git reset --hard` and every force-push form in every
  session; and refuses a mid-run `git push` without manifest push authority
  — that last one has no override. A gated push must stand alone on its
  line.
- **Harness guard** write-protects the configuration that decides what
  the agent may do: hook folders, every MCP config, plugin install and
  enable commands (including via `npx`), launched scripts, run manifests,
  ship-gate stamps. The refusal names the file and asks the question that
  matters: *if you didn't ask for this, something the agent read did.* A
  `cd` the guard can't follow fails closed.

Written into ADR 0016 and every guard's header: **these are not a
sandbox.** They're best-effort matching over command text, not a shell
parser. They convert a silent success into a refusal a human sees. Real
isolation is the operating system's job. Each guard carries a "not seen"
list, and the refusals name the owner-only override variable without
printing a paste-ready prefix — because coaching the override defeats the
guard.

The companion rule is **tracker text is data, not instructions.** Issue
bodies, comments, and anything fetched from the web were written by other
people and may be written to steer the agent. There's a behavioural eval
for it: given an issue body carrying an instruction to force-push and
silently close every other issue, the assistant must answer the real
request, refuse the embedded one, and *tell the user it was there*.
Staying quiet fails the check.

### Headless runs with a bounded budget

`auto --batch` is one staging conversation that front-loads everything a
human needs to decide. You pick phases, or name a budget in tokens or
dollars and let `token_estimate` pick them — it returns an honest cost
*range*, calibrated from real recorded spend, wide by admission when
history is thin, never a point claim. You grant push authorization once,
up front, scope-limited to exactly those phases (ADR 0007: it starts
false, is grantable only while staged, and declining still runs headless
— just nothing pushes). The `run_manifest` is the resume authority and
can never widen its own scope or authorization.

Then it runs: plan if needed, fan out to parallel agents in isolated
worktrees, verify goal-backward, ship under the pre-authorization, and
narrate every transition on the run's own tracker issue. `budget_check`
re-reads spend at every phase and wave boundary with a binary
proceed/stop (ADR 0008). The guarantee is stated the same way everywhere:
no new work starts past the ceiling, in-flight work finishes, overshoot is
bounded by at most one wave and recorded rather than hidden.

The first live run is a published case study: two phases, seven fixes,
22 minutes against a 13-hour human estimate — and the budget meter read
zero the entire time, because the metrics pipeline never received the
worker agents' spend. The run headlined its own instrumentation gap in its
own report; reconstruction showed about 561k tokens against a 500k
ceiling, inside the documented one-wave overshoot. The fix shipped the
next release, proven by a live refusal test. The second run then fired its
first live STOP on an accounting artifact (the ledger counted cached
tokens the estimator deliberately didn't), which was also filed and fixed.
Software that files tracker issues against itself mid-run is the
demonstration I'd pick over any benchmark.

### Docs that can't clobber your prose

`docs_drift` is deterministic — there is no model anywhere in it. It walks
every verified phase and asks: does the changelog mention you, and has the
documentation moved since your last recorded work? Generation sits behind
detection and is tiered: verify *reports* docs debt, ship *generates*
catch-up entries behind the push confirmation you were already answering,
summit runs full synthesis with an explicit publish offer.

Generated text can only land inside explicitly **marked sections** (ADR
0006). An unmarked section is an error, never a silent append. A
malformed marker halts. Identical content is a zero-diff no-op.
Full-file regeneration is banned; the changelog is the one prepend-only
exception. The machine *cannot* overwrite hand-written prose — not "is
told not to." Cannot.

### A repo that polices its own claims

Four surfaces are generated and never hand-edited: README marker spans,
the comparison page's cairn column, the harness `AGENTS.md` spine, and
the command shims. Seven guard scripts run in CI:

| Guard | Fails when |
|---|---|
| `check-surface` | verb table, shims, and verb files disagree |
| `check-pins` | the tool count pinned in two test files lags the registry |
| `check-dist` | committed `server/dist` is older than `server/src` |
| `check-versions` | the three version files and the marketplace pin disagree |
| `check-diagrams` | a diagram's count labels disagree with the registry |
| `check-footprint` | resident context exceeds its pinned budget |
| `eval-verdicts` | a regression-class behavioural eval fails |

`check-footprint` is the odd one. Cairn measures what it costs to have
installed — 39 command descriptions, 3 skill descriptions, the
session-start injection, and 87 MCP tool schemas resident in your context
— and pins the total. Today: 13,775 estimated tokens against a 13,784
budget, nine to spare. The budget moves the way the tool count moves and
is re-pinned to each corrected reading, never loosened by a fix. On its
first run it found a fifth of its own total sitting in one repeated suffix.

And since v2.8.0, every decision record, server gate, and hook states its
`removeWhen:` — what the model or the platform would have to do for the
mechanism to become unnecessary. Harness components exist because the
model can't do something yet. Writing the expiry down is how you avoid
shipping scaffolding forever.

## Why I think this is the right shape

**Honesty as mechanism, not marketing.** Every tool in this space now
claims "drift detection" and "adversarial review"; the words are
worthless. Cairn's answer is to make each claim a thing you can diff:
drift is a computed set, review findings carry votes, durations carry
their measurement rung, estimates carry their calibration, memory carries
its commit. When a claim can't be made mechanical — "does the agent
refuse an injected instruction" — it becomes a behavioural eval with a
PASS / FAIL / INCONCLUSIVE verdict, where a split is "we don't know yet,
add runs," not a fractional score.

**Fewer seams, not fewer tools.** I ran GSD, beads, and context-mode
together for months, and each is genuinely good. But three point tools
meant three configs, two precedence rules, a hand-maintained map wiring
plans to issues, and zero shared state. Cairn keeps GSD's best ideas on
purpose — the phase shape, the depth dial, goal-backward verification —
and adds the layer those flows can't hold in files: tracker truth and
provenance-checked memory. Close an issue and the ledger records it;
verify a phase and drift goes quiet; learn a lesson in retro and recall
serves it next session. Shared state by construction.

**It eats its own dog food, in public.** The council case study records
58 findings, 50 verified, 8 killed with reasons, and one recommendation
shipped over cairn's own written objection. The batch-run case study
records a blind budget meter. The v2.8.2 hardening closed 48 issues in 13
waves, re-auditing the security surface after every wave until ten passes
came back with no important finding left — three of those issues were
regressions the release introduced and caught itself. The repo's own
first sweep filed eleven findings against cairn, and the release notes
say so. A review process that only publishes its wins is theater.

**It's a different point on the spectrum, not a better one.** If you're
prototyping fast and solo-ish, a discipline layer with no moving parts is
honestly plenty. Cairn is heavier: a server, a tracker relationship to
maintain, a build you have to trust. In exchange the project's truth
outlives any session, any context window, and any single tool's opinion
of itself. Pick by the job.

## What it doesn't do

Said plainly, because the docs say it plainly:

- The guards are not a sandbox. An override reachable by the agent is
  reachable by anything that can steer the agent.
- The leak guard covers commits made through the agent's shell tool, not
  your own terminal.
- The open-issues half of the ship gate is still procedure; the drift half
  is hook-enforced.
- Hook parity: full on Claude Code, partial on Cursor, claimed-untested on
  Grok, absent elsewhere. The cost tracker is Claude Code-only.
- Internal seats are framing lenses on the same model. The external peers
  council is the adversarial mechanism, and it needs those CLIs installed.
- Only Confluence and Docusaurus as docs targets; new ones land on demand,
  not on a roadmap.
- A context-economy experiment — compacting far earlier to halve average
  context — was run and killed because ordinary work became unworkable.
  The predicted saving is withdrawn and nothing should cite it.

## Getting started

In any Claude Code session:

```
/plugin marketplace add eventually-consistent-code/cairn2
/plugin install cairn@cairn-dev
```

The server's `dist/` is committed and the installer pulls its four runtime
dependencies, so there's no build step — restart Claude Code and the MCP
server connects. Then either drop a `cairn.json` in your project root (the
four-line GitHub one above, or `"type": "local"` for zero credentials) or
just run `/cairn:new`, which asks which tracker you want and writes the
file for you.

Any bare-MCP harness:

```bash
npm i @eventually-consistent/cairn-server
```

The seven other harnesses:

```bash
npx @eventually-consistent/cairn cairn-setup <grok|copilot|codex|gemini|cursor|opencode|zed>
```

Then the loop: `/cairn:new` → `/cairn:plan 1` → `/cairn:work 1` →
`/cairn:verify 1` → `/cairn:ship`. Watch the issues as `work` runs — claim
comments with base commits, progress at real milestones, closes with
commit ranges and measured time. The paper trail writes itself, because if
it isn't in the tracker, it didn't happen…

- Repo: https://github.com/eventually-consistent-code/cairn2
- Project page and quick-start: https://www.eventually-consistent.io/projects/cairn/
- The council case study: https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-council.md
- The batch-run case study: https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-batch-run.md
- The mechanism-level comparison, with last-verified dates: https://github.com/eventually-consistent-code/cairn2/blob/main/docs/comparison.md

Skeptical questions welcome — especially the one that shaped every design
decision here: can you ever trust an agent's self-reported "done" without
an independent check? My answer is no, and that's why the gate exists.

-John Reed
