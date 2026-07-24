+++
title = "Cairn"
date = 2026-07-24T00:00:00Z
draft = true
tags = ["claude-code", "agentic-tooling", "project-management", "developer-tooling"]
summary = "A Claude Code plugin that plans your project, mirrors every piece of work to your real issue tracker, remembers what it learned, and verifies what it shipped — one verb set instead of a stack of point tools."
repo = "https://github.com/eventually-consistent-code/cairn2"
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
testing it. 36 verbs, 63 typed MCP tools, six tracker backends — GitHub,
GitLab, Jira, Asana, Azure Boards, ClickUp — 495 passing tests, three
runtime dependencies.

The name is the point. A cairn is a stack of stones that marks the trail for
whoever comes next — including future you, who has forgotten everything…

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

## What that looks like in practice

**Your tracker gets a paper trail, not a paste-dump.** When cairn works an
issue, the issue shows it: a claim comment with the base commit, progress
comments at real milestones, and a close comment with the commit range, test
evidence, and time spent — logged as a real worklog on backends that support
it. This page's own Jira issue closed with "hugo builds clean, 0 errors;
nav order confirmed; time spent: ~15m." Dogfooding is the demo.

**Your PM edits an issue; cairn notices.** `plan_tracker_delta` diffs the
live tracker against a cursor: new issues, edited bodies, state changes made
outside the session. `/cairn:resync` ingests them instead of discovering
them mid-flight. Planning tools that only push are half a mirror.

**Memory admits when it's out of date.** `/cairn:remember` writes a
git-committed card — one fact, with provenance: which file, which commit,
which decision produced it. When the source moves, `/cairn:recall` serves it
flagged STALE instead of confidently reciting fiction. The rule is blunt:
memory can be wrong; it can never silently lie.

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

## The matrix

The honest version, not the checkbox-farming version — each of these tools
is good at its job, and the footnotes carry the nuance.

| Capability | cairn | GSD | beads | context-mode |
|---|---|---|---|---|
| Phase planning with research depth control | ✓ | ✓ | — | — |
| Tracker mirroring — six backends, both directions | ✓ | partial¹ | partial² | — |
| Comment + worklog paper trail on issues | ✓ | — | —³ | — |
| Inbound PM-delta ingest (tracker edits flow back) | ✓ | — | — | — |
| Memory with provenance + staleness flags | ✓ | — | — | partial⁴ |
| Goal-backward verification + drift gate | ✓ | partial⁵ | — | — |
| Multi-project workspaces | ✓ | ⁶ | — | — |
| Persistent debug/spike/design sessions | ✓ | — | — | — |
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

## Shipped here, requested there

The matrix rows aren't hypothetical gaps — they're things users of each tool
asked for, in public, often more than once. A sampling, straight from their
own trackers:

**What beads users asked for:**

- Pluggable tracker integrations — Jira, Azure DevOps, GitLab
  ([#1150](https://github.com/gastownhall/beads/issues/1150), closed *not
  planned*) and more backends by name
  ([Linear](https://github.com/gastownhall/beads/discussions/453), the
  category's top-voted idea). Cairn: six adapters behind one interface.
- Work-time tracking
  ([#965](https://github.com/gastownhall/beads/issues/965), closed *not
  planned*). Cairn: time logged on every close, real worklogs where the
  backend supports them.
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
