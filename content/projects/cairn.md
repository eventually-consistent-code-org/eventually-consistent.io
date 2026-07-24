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

That's the problem cairn exists to solve. It's a Claude Code plugin that gives
an agent the things a real project needs: a plan that lives in git, work items
that live in your actual tracker, memory that survives the session, and a
verification gate that refuses to call something done because closing the
issue was easier than testing it.

The name is the point. A cairn is a stack of stones that marks the trail for
whoever comes next — including future you, who has forgotten everything…

## The lifecycle

You start with `/cairn:new`. It interviews you — what are you building, what
does done look like — and writes the answers down where they can't evaporate:
a `PROJECT.md` with requirements, a roadmap of phases, a context file per
phase for decisions that get locked. Then it mirrors the whole thing into
your tracker. Phases become epics or milestones, requirements become issues,
and from that moment the tracker your manager reads and the plan the agent
executes are the same thing.

`/cairn:plan N` researches a phase before touching it — how deep is a dial
you control, from "just draft the tasks" to a parallel research fan-out with
an adversarial plan-checker pass. `/cairn:work N` claims each issue, does the
work, comments progress as it lands, and closes on verified done — tests
passing, behavior confirmed, never "the gate wanted it closed."
`/cairn:verify N` then walks the phase backwards from the goal: not "did the
tasks close" but "does the codebase deliver what the phase promised." Only
then does `/cairn:ship` push — and it refuses if the plan and tracker have
drifted apart.

Thirty-six verbs total, and they cover the unglamorous parts too: `trace` for
debugging sessions that persist evidence and hypotheses, `probe` for
throwaway spikes with recorded verdicts, `retro` for lessons that become
memory instead of vibes, `medic` when the planning directory itself needs a
doctor. All of it backed by 63 typed MCP tools on a small local server —
three runtime dependencies, 495 passing tests.

## The tracker is not an afterthought

Most agent tooling treats your issue tracker as a place to eventually paste a
summary. Cairn treats it as a first-class mirror: six backends — GitHub,
GitLab, Jira, Asana, Azure Boards, ClickUp — behind one normalized interface,
with one contract test suite every adapter has to pass.

The mirroring runs both directions. Everything the agent does leaves a paper
trail a manager can read: claim comments with base commits, progress comments
at real milestones, close comments with commit ranges, test evidence, and
time spent — logged as an actual worklog on backends that support it. And
when your PM edits an issue or drops a new one in the backlog, `resync`
ingests the delta instead of discovering it the hard way.

One rule keeps the whole thing sane: on conflict, the git plan docs win over
tracker issue text. The agent updates the issue with a dated note — it never
silently follows a stale ticket, and it never silently ignores one either.

## Memory that can be wrong but can never lie

Cairn's memory has two tiers. Tier one is a disposable full-text index that
lives outside your repo — fast recall, zero ceremony. Tier two is the one
that matters: git-committed memory cards, one fact each, with provenance —
which file, which commit, which decision produced it. When the source a card
points at moves or changes, recall flags it STALE instead of confidently
reciting fiction. The design rule is blunt: memory can be wrong; it can never
silently lie.

Continuity gets the same treatment. Session handoffs survive `/clear`,
compaction, and crashes; `waypoint` pauses a session and resumes it days
later; threads keep long-running context alive across restarts. The agent
you talk to tomorrow knows what the one you talked to today was doing…

## The parts nobody markets

Verification is where agent work usually falls apart, so cairn is strict
there: goal-backward phase checks, a five-axis code review verb, audit modes
for UAT, security, UI fidelity, and test quality, and an append-only ledger
recording what shipped, in which commits, closing which issue. Failure is
reported honestly — a failed verify is a failed verify, and the design
principle underneath it all is *fail loud, never fake state*. Errors surface
as typed codes with a next action, never a stack trace, and one backend being
down never blocks git-side work.

It scales past one repo, too: `basecamp` runs multi-project workspaces with a
dispatch board, `peers` convenes external AI CLIs as reviewers with an
outbound leak guard, and an engineer mode turns the agent from a solo
performer into a pairing partner — it stops at genuine design forks, hands
you the trade-offs, and never merges its own PRs.
