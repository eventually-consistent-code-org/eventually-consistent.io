+++
title = "Cairn"
date = 2026-07-24T00:00:00Z
draft = false
tags = ["claude-code", "agentic-tooling", "project-management", "developer-tooling"]
summary = "An agent project brain that plans your project, mirrors every piece of work to your real issue tracker, remembers what it learned, and verifies what it shipped — born a Claude Code plugin, now running on Grok, Copilot, Codex, Gemini, Cursor, OpenCode, and Zed too."
repo = "https://github.com/eventually-consistent-code/cairn2"

[cover]
image = "img/cairn-logo-h.jpg"
alt = "Cairn logo — a stack of colorful glass pebbles"
hiddenInList = true
+++

Ever asked an AI agent to build something real — not a demo, a project with
phases and a deadline and a boss who asks how it's going — and watched the
plan evaporate the moment the context window rolled over? The agent was
brilliant for an afternoon and amnesiac by morning. The plan lived in a chat
scrollback. The issue tracker your team actually looks at never heard about
any of it.

Cairn fixes that. It's a Claude Code plugin that gives an agent what a real
project needs: a plan that lives in git, work items that live in your actual
tracker, memory that survives the session, and a verification gate that
refuses to call something done because closing the issue was easier than
testing it. 37 verbs, 71 typed MCP tools, eight tracker backends — GitHub,
GitLab, Jira, Asana, Azure Boards, ClickUp, Linear, and a zero-credential
local tracker that lives in your repo — 744 passing tests, three runtime
dependencies. And as of this week it's no longer Claude-only: one installer
wires the same brain into Grok Build, Copilot CLI, Codex, Gemini CLI,
Cursor, OpenCode, and Zed.

The name is the point. A cairn is a stack of stones that marks the trail for
whoever comes next — including future you, who has forgotten everything…

The code is public:
[github.com/eventually-consistent-code/cairn2](https://github.com/eventually-consistent-code/cairn2).

## The lifecycle in five commands

```
/cairn:new       interview → PROJECT.md, roadmap, phases → mirrored to your tracker
/cairn:plan 2    research the phase, write the task breakdown, reconcile drift
/cairn:work 2    claim issues, do the work, close on verified done
/cairn:verify 2  goal-backward check: does the code deliver what the phase promised?
/cairn:ship      refuses unless drift-clean and every verified phase is closed out
```

Phases become epics or milestones. Requirements become issues. From the first
command, the tracker your manager reads and the plan the agent executes are
the same thing.

Here's the whole shape at a glance — every verb touches both lanes at once,
and the mirror runs in both directions:

![the cairn workflow — git plan artifacts above the verb spine, the tracker paper trail below, with the resync channel running back the other way](/img/cairn-workflow.png)

## What that looks like in practice

**Your tracker gets a paper trail, not a paste-dump.** When cairn works an
issue, the issue shows it: a claim comment with the base commit, progress
comments at real milestones, and a close comment with the commit range, test
evidence, and time spent — logged as a real worklog on backends that support
it. This page's own Jira issue closed with "hugo builds clean, 0 errors;
nav order confirmed; time spent: ~15m." Dogfooding is the demo.

**Your docs land where the team already reads.** `/cairn:docs publish`
mirrors the repo's README and `docs/` folder into Confluence: a folder named
for the project, the README as its landing page with a generated table of
contents, and the whole doc tree as child pages underneath. Re-publishing
updates pages in place — never duplicates. The connector interface is
pluggable, so Notion, GitBook, Slite, and SharePoint can follow behind the
same contract; and yes, the feature's first live run published cairn's own
43-page doc tree.

**No tracker? No problem — start local, graduate later.** The built-in
local backend stores issues as plain files in your repo: no accounts, no
credentials, merge-clean by construction, and a real dependency graph —
typed links between issues, a ready-to-work frontier, priority that
inherits through blocking chains (a P3 blocking a P1 *is* a P1). When the
team grows, `tracker_migrate` promotes the whole store to Jira, GitHub, or
any other backend in one command — history, comments, links, and estimates
carried over with provenance backlinks.

**Your sprint reports stop being empty.** On Jira scrum boards, cairn
detects the board, drops new issues into the active sprint, and populates
story points + original time estimates at planning time — then logs real
worklogs at close. Burndown, velocity, and estimate-vs-actual work with
data cairn wrote, not fields a human forgot to fill.

**You see what the agent costs, per issue.** Every session's token spend
gets snapshotted and attributed to the phase and issue that was active —
so close comments carry "agent cost: ~$4 (approximate)" beside time spent,
and status rolls spend up by phase. Estimate vs actual vs spend, on the
board your manager already reads. No other tool in this lane attributes
agent cost to work items.

**Your board's words, not cairn's.** Issues carry their real workflow
state — "In Review", "Blocked", whatever your columns actually say —
with a semantic category underneath so every gate and report still knows
what counts as done. Custom states are one config line per backend; Jira
fires your actual transitions, Linear matches your team's workflow states
by name.

**Evidence attaches where the team looks.** Screenshots and visual
findings from the UI audits attach to the tracker issue itself (native
Jira attachments, files-in-repo on the local backend), and images in your
published docs upload as real Confluence attachments that render inline —
architecture diagrams included, never broken links.

**Your PM edits an issue; cairn notices.** `plan_tracker_delta` diffs the
live tracker against a cursor: new issues, edited bodies, state changes made
outside the session. `/cairn:resync` ingests them instead of discovering
them mid-flight. Planning tools that only push are half a mirror.

**Memory admits when it's out of date.** `/cairn:remember` writes a
git-committed card — one fact, with provenance: which file, which commit,
which decision produced it. When the source moves, `/cairn:recall` serves it
flagged STALE instead of confidently reciting fiction. The rule is blunt:
memory can be wrong; it can never silently lie. And it learns while you
work: a passive observation log quietly notes what each tool call touched
and whether it errored, and at retro time the real patterns — churn
hotspots, repeated error-fix loops — get promoted to cards with
provenance. Nothing enters memory without that review gate.

**Debugging is a session, not a scrollback.** `/cairn:trace` opens a
persistent evidence → hypothesis → test session, mirrored to a tracker issue
so the investigation is visible work, not invisible flailing. It survives
`/clear`. So do `probe` (throwaway spikes with recorded verdicts), `draft`
(design variants), and `thread` (long-running context).

**Verification walks backwards from the goal.** Not "did the tasks close" —
"does the codebase deliver what the phase promised." Five-axis code review,
audit modes for UAT, security, and UI fidelity, and an append-only ledger of
what shipped in which commits. A failed verify is a failed verify; *fail
loud, never fake state* is a design principle, not a slogan.

**It scales past one repo.** `basecamp` runs multi-project workspaces with a
dispatch board. `peers` convenes external AI CLIs as reviewers behind an
outbound leak guard. Engineer mode turns the agent into a pairing partner —
it stops at genuine design forks, hands you the trade-offs, and never merges
its own PRs.

## Runs everywhere your agent does

Why should a project brain care which agent is driving? It shouldn't — and
that's the architecture doing the work, not a porting marathon. Cairn's
server speaks plain MCP over stdio, and every verb is a markdown
subroutine any capable agent can read and follow. So the "port" to a new
harness is three thin layers: point the harness's MCP config at the
server, install the generated `AGENTS.md` verb registry, and add a slash
surface where the harness has one.

One installer does all three:

```
node setup/cairn-setup.mjs grok      # or copilot, codex, gemini,
                                     # cursor, opencode, zed
```

Copilot gets `/cairn-*` prompt files, Codex gets prompts plus a managed
`config.toml` block, Gemini gets `/cairn:*` TOML commands with the exact
Claude Code spelling, and Cursor gets the part nobody else has tried: a
real port of the hook layer. One adapter translates Cursor's hook events
into the shape cairn's hook scripts already speak — so continuity resume,
the commit leak guard, and observation capture all work in Cursor,
unchanged. The honest footnote: the cost tracker stays Claude Code-only
(it reads Claude's transcript format for token counts), and that's
documented per-harness rather than hand-waved. The tracker paper trail
doesn't care which model wrote it…

## The matrix

The honest version, not the checkbox-farming version — each of these tools
is good at its job, and the footnotes carry the nuance.

| Capability | cairn | GSD | beads | context-mode |
|---|---|---|---|---|
| Phase planning with research depth control | ✓ | ✓ | — | — |
| Tracker mirroring — eight backends, both directions | ✓ | partial¹ | partial² | — |
| Zero-dependency local tracker with dependency graph | ✓ | — | ✓ | — |
| Local → hosted promotion (full history migration) | ✓ | — | — | — |
| Sprint + estimate population (real burndown data) | ✓ | — | — | — |
| Agent cost attributed to issues/phases | ✓ | — | — | — |
| Custom status vocabulary (your board's real states) | ✓ | — | — | — |
| Comment + worklog paper trail on issues | ✓ | — | —³ | — |
| Inbound PM-delta ingest (tracker edits flow back) | ✓ | — | — | — |
| Memory with provenance + staleness flags | ✓ | — | — | partial⁴ |
| Goal-backward verification + drift gate | ✓ | partial⁵ | — | — |
| Multi-project workspaces | ✓ | ⁶ | — | — |
| Persistent debug/spike/design sessions | ✓ | — | — | — |
| Docs publishing to the team wiki (Confluence) | ✓⁸ | — | — | — |
| Runs beyond Claude Code (7 more harnesses) | ✓ | — | — | — |
| Context-window economy | — | — | — | ✓⁷ |

¹ GSD shipped one-way GitHub sync after sustained user demand
([#2191](https://github.com/gsd-build/get-shit-done/issues/2191)); one
backend, push only.
² beads syncs to GitHub/GitLab/Jira, but its own tracker reports fidelity
gaps — comments don't sync
([#4452](https://github.com/gastownhall/beads/issues/4452)), IDs and
dependencies get lost
([#4307](https://github.com/gastownhall/beads/issues/4307)) — and a
plugin-based tracker integration architecture was closed *not planned*
([#1150](https://github.com/gastownhall/beads/issues/1150)).
³ Work-time tracking was requested and closed *not planned*
([#965](https://github.com/gastownhall/beads/issues/965)).
⁴ context-mode indexes and recalls, but staleness is its open wound: stale
cross-session memory outranking fresh captures is an open issue
([#895](https://github.com/mksglu/context-mode/issues/895)); silent
index-staleness needed a fix
([#317](https://github.com/mksglu/context-mode/issues/317)).
⁵ GSD has a verifier; cairn adds the tracker cross-check — drift detection,
no-open-issues gates, and an append-only ledger tying issues to commits.
⁶ Requested at least four separate times
([#108](https://github.com/gsd-build/get-shit-done/issues/108),
[#648](https://github.com/gsd-build/get-shit-done/issues/648),
[#1241](https://github.com/gsd-build/get-shit-done/issues/1241),
[#2033](https://github.com/gsd-build/get-shit-done/issues/2033)).
⁷ Its actual job, and it's good at it — see the verdict below.
⁸ `/cairn:docs publish` — folder-per-project, README as landing page,
generated TOC, full child-page tree, idempotent re-publish; pluggable SPI
with Notion/GitBook/Slite/SharePoint as planned adapters. Its first live
run published cairn's own 43-page doc tree.

## Shipped here, requested there

The matrix rows aren't hypothetical gaps — they're things users of each tool
asked for, in public, often more than once. A sampling, straight from their
own trackers:

**What beads users asked for:**

- Pluggable tracker integrations — Jira, Azure DevOps, GitLab
  ([#1150](https://github.com/gastownhall/beads/issues/1150), closed *not
  planned*) and more backends by name
  ([Linear](https://github.com/gastownhall/beads/discussions/453), the
  category's top-voted idea). Cairn: eight adapters behind one interface —
  Linear included, with native issue links.
- Work-time tracking
  ([#965](https://github.com/gastownhall/beads/issues/965), closed *not
  planned*). Cairn: time logged on every close, real worklogs where the
  backend supports them — and agent cost attributed alongside.
- Custom status vocabularies
  ([#345](https://github.com/gastownhall/beads/discussions/345)). Cairn:
  issues carry your board's real state names with a semantic category
  underneath; custom states are one config line per backend.
- Image evidence on issues
  ([#875](https://github.com/gastownhall/beads/discussions/875)). Cairn:
  native attachments on Jira, files-in-repo on the local backend, wired
  into the UI audits.
- A spec layer above issues
  ([#240](https://github.com/gastownhall/beads/discussions/240)), human
  approval gates for enterprise oversight
  ([#632](https://github.com/gastownhall/beads/discussions/632)), a durable
  home for accreted decisions
  ([#611](https://github.com/gastownhall/beads/discussions/611)), and
  outcome-enforced closes instead of low-information "done"
  ([#856](https://github.com/gastownhall/beads/discussions/856)). Cairn:
  phase plans above the tracker, engineer-mode gates, provenance-backed
  memory cards, and verified-done closes with evidence.

**What GSD users asked for:**

- Issue-tracker integration
  ([#907](https://github.com/gsd-build/get-shit-done/discussions/907)) and
  plan state visible to "stakeholders, collaborators, and the dev's own
  memory weeks later"
  ([#2191](https://github.com/gsd-build/get-shit-done/issues/2191)). Cairn:
  the bidirectional mirror is the core of the design.
- Multi-project support, four separate times
  ([#108](https://github.com/gsd-build/get-shit-done/issues/108),
  [#648](https://github.com/gsd-build/get-shit-done/issues/648),
  [#1241](https://github.com/gsd-build/get-shit-done/issues/1241),
  [#2033](https://github.com/gsd-build/get-shit-done/issues/2033)) — plus
  parallel session dispatch
  ([#2621](https://github.com/gsd-build/get-shit-done/discussions/2621),
  [#3256](https://github.com/gsd-build/get-shit-done/discussions/3256)).
  Cairn: basecamp workspaces with a dispatch board; wave-parallel work.
- Retro and lessons that outlive the phase
  ([#1003](https://github.com/gsd-build/get-shit-done/discussions/1003),
  [#1873](https://github.com/gsd-build/get-shit-done/issues/1873),
  [#1680](https://github.com/gsd-build/get-shit-done/issues/1680)),
  autonomous UAT
  ([#2290](https://github.com/gsd-build/get-shit-done/discussions/2290)),
  resync against out-of-band changes
  ([#944](https://github.com/gsd-build/get-shit-done/discussions/944)), and
  keeping planning internals out of shipped code
  ([#2221](https://github.com/gsd-build/get-shit-done/discussions/2221)).
  Cairn: `retro`, `audit uat`, `resync`, and the leak guard — all verbs, all
  shipped.

**What context-mode users are living with:**

- Cross-session persistence was asked for early
  ([#250](https://github.com/mksglu/context-mode/issues/250),
  [#305](https://github.com/mksglu/context-mode/issues/305) — closed *not
  planned*), and staleness is the open wound: stale memory outranking fresh
  work ([#895](https://github.com/mksglu/context-mode/issues/895), open),
  continuity snapshots silently losing the newest events
  ([#902](https://github.com/mksglu/context-mode/issues/902)). Cairn's
  memory cards are cross-session by construction with staleness flags by
  design, and its checkpoints are explicit durable artifacts — not derived
  from a buffer that evicts.

## The verdict on the stack

I ran GSD + beads + context-mode together for months, and each one is
genuinely good. But three point tools means three configs, two precedence
rules, a hand-maintained map file wiring plans to issues, and zero shared
state — the planner doesn't know what the tracker knows, and neither knows
what memory knows.

Cairn keeps GSD's best ideas (the phase shape, the depth dial, goal-backward
verification), replaces beads with the tracker your team already reads, and
covers the project-memory slice of context-mode with provenance instead of
retrieval. context-mode still earns a companion slot for raw context
economy — that's its job, and cairn doesn't pretend to do it.

One plugin, one verb set, shared state by construction: close an issue and
the ledger records it, verify a phase and drift goes quiet, learn a lesson
in retro and recall serves it next session. Fewer tools isn't the point.
Fewer seams is…

Ready to try it? The [quick-start](/projects/cairn-quickstart/) goes from
install to a tracker-mirrored first project in about fifteen minutes —
including every step that happens outside Claude.
