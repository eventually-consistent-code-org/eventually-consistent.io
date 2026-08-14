+++
title = "The Work Harness: What Cairn Became, and the Competitor It Grew Up Into"
date = 2026-08-14T00:00:00Z
draft = false
tags = ["cairn", "claude-code", "agentic-tooling", "gsd", "workflow", "ai", "meta"]
summary = "Six weeks of shipping turned cairn from a wrapper around other people's tools into a full work harness — a four-vendor AI review council, a portfolio board, a platform-native rebuild, and the honest answer to the question that review surfaced: who is cairn actually competing with?"
+++

Here's a question worth asking before you install *any* AI coding tool:
what kind of harness is it?

Because that's what these things are — harnesses. Something you strap
around an AI so the horsepower goes somewhere useful. And they run the
gamut. At one end you've got the skeletal ones: a loop, a prompt, get
code out the door, godspeed. Nothing wrong with that — sometimes you
want a fast horse and no saddle. In the middle sit the discipline
layers: skills, checklists, phase files, conventions that make the
agent *behave* better without holding any state you'd miss if it
vanished. And at the far end sit full work harnesses — systems that
hold the plan, the work items, the memory, and the verification, so the
agent can carry a real project across weeks without dropping it.

Cairn lives at that far end, on purpose, and this post is the story of
the six weeks that put it there — what shipped, why, and the slightly
uncomfortable thing an AI review council told me about who my actual
competitor is…

## Why I care about this end of the spectrum

I'm a solo developer. There's no standup to surface what's
half-finished, no teammate holding the project's shape in their head
while I sleep. A year ago that meant every real project cost months,
and most of those months weren't creative work — they were minutia.
Tracking what's done. Remembering why I decided something. Checking
whether the thing I closed actually works. The tax you pay just to keep
a project *organized*, and it compounds until the project quietly dies.

What a work harness buys me is exactly that tax back. The plan lives in
git. Every piece of work mirrors to a real issue tracker as a side
effect of doing it. Memory carries provenance, so a stale fact flags
itself instead of lying to me. Verification is a gate, not a vibe. And
the loop stays *fast* — I describe intent, review, steer, and the
creative part of my brain drives while the harness sweeps up behind me.
I've built things this year I flatly could not have built alone before,
and it's not because the AI got smarter. It's because nothing leaks
anymore.

If I had a team, cairn's tracker mirror would be the collaboration
story — a manager reads GitHub or Jira and sees everything, in plain
language, without asking. Working solo, the beauty is simpler: it's the
teammate I didn't have.

## What actually shipped — and the why behind each piece

Since the [last post](/posts/four-tools-three-weeks/), cairn went from
version 2.0 to 2.3+, from 744 tests to 1,117, from 71 tools to 79. The
raw changelog is long; the *why* comes in five moves.

**First, the pipe got trustworthy.** A stranded release taught me that
publishing and installing are part of the product — users had a version
whose server couldn't even start, and nothing anywhere said so. So now:
one release command bumps every version surface atomically, CI fails if
the three version files disagree, the marketplace pins exact releases,
and installs can self-report "installed vX, available vY." Unsexy work.
It paid for itself within a week — more on that below.

**Second, cairn got reviewed by a jury of its rivals.** The `peers
council` convened AI reviewers from four competing vendors — codex,
grok, antigravity, opencode — sixteen seats, same screened evidence,
one rule: a claim without a locatable evidence ref is discarded before
convergence. 58 findings, 50 verified, 8 killed with reasons. The
council caught a real security gap in its own process mid-run, and one
recommendation shipped *over cairn's own written objection* — the tool
argued no-action, the reviewers disagreed, I sided with the reviewers,
and the record says so. The whole thing is public now as a
[case study](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-council.md),
kills and outages and all, because a review process that only publishes
its wins is theater.

**Third, the portfolio view.** New `outlook` verb: every cairn project
on the machine keeps a compact status snapshot as a side effect of
normal work, and one command renders the fleet — per-project cards,
freshness flags computed against git, cost totals split by work type,
approximate and labeled as such. Solo-dev honesty: I have more projects
than attention, and the board tells me where the stale ones are without
opening a single repo.

**Fourth, cairn went native.** The platform under it moved fast this
year, and the right response to a platform shipping primitives is to
build *on* them, not beside them. So cairn migrated to the new MCP v2
protocol stack, taught Claude Code's built-in task list to mirror into
your real tracker automatically (a task created in-session becomes a
tracker item; finishing it closes the item — hooks, no polling), and
rewired its parallel-work mode onto the platform's own workflow engine,
with resume that picks an interrupted batch up where it stopped. Each
of those started life as a throwaway probe with a written verdict
before a line of production code — burn the riskiest assumption first,
keep the receipt.

**Fifth, proof over pitch.** Every tool in this space now claims "drift
detection" and "adversarial review," so those words are worthless. The
repo now carries a
[mechanism-by-mechanism comparison page](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/comparison.md)
where every competitor claim has a *last-verified date*, plus the case
study above. Artifacts beat announcements.

And that release-integrity work from move one? Hours after the v2
migration merged, the version-sync gate went red because the migration
updated the server's dependency list but missed the copy that installed
plugins actually consume — the exact class of failure that once shipped
a dead install. The gate caught it on its first real opportunity. Two
line fix, green build. That's what "the pipe is part of the product"
means in practice.

## The competitor the council named — and the honest comparison

The council's market panel didn't hedge: cairn's public framing was
aimed at the wrong rivals, and the real substitute is **GSD** and its
lineage (GSD Pro, the open-gsd fork, buildomator).

Here's the part I love. Cairn *started as a GSD wrapper*. The
[June post](/posts/wiring-gsd-and-beads/) on this very blog is me
gluing GSD, beads, and context-mode together because each held a piece
of what I needed. The wrapper grew a server, then a tracker layer, then
memory, then verification, and somewhere along the way it stopped
wrapping and started competing. So this comparison isn't
drive-by — I ran GSD daily for months, and cairn still carries its best
ideas on purpose: the phase shape, the depth dial, goal-backward
verification. Credit where due, permanently.

The honest breakdown, mechanism by mechanism:

**Where GSD is genuinely strong.** It's huge — roughly 60k stars — and
that community gravity is real: more eyes, more patterns, more Reddit
answers at 2 a.m. It's file-based, which travels beautifully across 8+
harnesses with zero server to run. Its discuss→plan→execute→verify→ship
cycle is a genuinely good shape (cairn's phase model rhymes with it,
and that's not an accident). If you want a discipline layer with no
moving parts and a giant community, GSD is a fine answer — I will not
pretend otherwise.

**Where the mechanisms diverge.** Everything below is on the dated
comparison page; the short version:

- *Where work truth lives.* GSD's truth is repo files; it added one-way
  GitHub sync after sustained user demand — an export. Cairn's truth is
  your external tracker, with eight write-through backends (GitHub,
  GitLab, Jira, Asana, Azure Boards, ClickUp, Linear, a zero-credential
  local one). The tracker your team — or your manager, or future
  you — already reads *is* the record.
- *What "drift" actually computes.* In the GSD lineage, drift means
  internal state drift — handoff checkpoints, worktree staleness.
  Cairn's drift is plan↔external-tracker math: the server diffs what
  the plan claims against what the tracker says and blocks shipping on
  the difference. Same word, different mechanism, and the word alone
  no longer tells you which one you're getting.
- *Verification.* GSD has a verifier for its own plan steps. Cairn's
  verify gate works goal-backward — did the codebase deliver what the
  phase *promised* — and a closed-but-unverified issue is a flag, not
  a success.
- *Review.* GSD Pro routes across models to generate. Cairn's council
  seats four rival vendors to *judge*, and claims die without evidence
  refs. Generation routing and adversarial verification are different
  machines wearing similar marketing.
- *Memory.* GSD's state files carry no provenance. Cairn's memory cards
  record the file and commit they came from, and recall re-checks that
  provenance against git and flags `STALE` instead of asserting a dead
  fact.

None of that makes cairn "better" in the abstract — it makes it a
different point on the harness spectrum. GSD is a great discipline
layer. Cairn is a work harness: heavier, with a server and a tracker
relationship to maintain, and in exchange the project's truth outlives
any session, any context window, and any single tool's opinion of
itself.

Pick by the job. If you're prototyping fast and solo-ish, skeletal or
discipline-layer is honestly plenty. If you're building something with
phases and a deadline and a future — or you're one person doing the
work of a team and you need the team's memory without the team — that's
what the far end of the spectrum is for…

## Try it

```
/plugin marketplace add eventually-consistent-code/cairn2
/plugin install cairn
```

or `npm i @eventually-consistent/cairn-server` for any bare-MCP harness
(fresh installs are smoke-tested from the public registry — the pipe is
part of the product). Start with
[the quickstart](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/00-quickstart.md),
and if you only read one artifact, read the
[council case study](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-council.md) —
it's the most honest thing I've ever published about my own software.

-John Reed
