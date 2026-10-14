+++
title = "The Community Is the Design Review: Six Articles, Four Phases, One Night"
date = 2026-10-14T00:00:00Z
draft = true
tags = ["cairn", "claude-code", "agentic-tooling", "community", "open-source", "ai", "meta"]
summary = "One evening on dev.to turned into four new phases on cairn's roadmap, three new global rules on my machine, and four issues shipped the same week. The mechanism was reading other people's write-ups as bug reports against my own code. Here's the loop, the receipts, and the people to thank."
+++

Ever sit down to "catch up on reading" and look up three hours later with
your roadmap rearranged?

That was October 13th for me. I'd spent two months heads-down on
[cairn](/projects/cairn/), the work harness I build for AI coding agents,
shipping release after release with my test suite as the only thing
reviewing my design. A test suite is a fine reviewer for "does it work"
and a hopeless one for "should it be shaped like this at all." So I opened
dev.to to decompress, read six articles by six people I've never met, and
by the end of the night cairn had phases 27 through 30 on its roadmap,
four issues merged inside the week, and every project on my machine had
three new rule files governing how the agent behaves in it…

This post is about the loop that got me there, because I think the loop
is the most underrated thing about building in the open right now, and
the specifics are mostly here to prove it ran.

## The loop

Read the article whole. Then, before you form an opinion, go check your
own tree for the thing it describes. Not "does this sound right" but "does
my code actually do this, and at which line." If the author is right about
you, file the fix as a phase with the article named in its `## Source`
section. If the author is wrong about you, or your grain already matches,
write that down too, because a confirmed decision with a citation is worth
nearly as much as a new one. Then pay it forward: attribution in every
file that borrows the idea, paraphrase the reasoning, copy no blocks.
Clean-room reading, with receipts.

The grounding step is the whole trick. An idea you admire is worth nothing
to your project until you've found the line where you were wrong.

## What walked out, article by article

### A constitution, not a prompt

Alex Aslam's piece argues that rules written per incident never converge.
Each prohibition leaves the adjacent behavior uncovered and you stay one
failure behind forever, so what you need is a classification scheme that
makes the *next* unseen case recognizable.

I opened cairn's run guard. Its header listed seventeen issue numbers.
Seventeen times something went wrong, seventeen rules each justified by the
bug that caused it. The article was describing my code, so it became
**phase 28, the-constitution**: five to seven axioms in the architecture
doc classifying what counts as what (the user's data versus scratch, a
destructive action, a directive versus data, done versus claimed, the
owner's checkout, the harness's own configuration). Every guard and every
ADR names the clause it derives from. A rule that derives from no clause
is itself the finding. The run guard gets rewritten to classify instead of
enumerate, with the seventeen existing tests kept green as the regression
corpus.

Two mechanisms came along for the ride. The server can re-run the
owner-declared `verify:` line at close and report observed-versus-claimed,
and headless runs get a circuit breaker; interactive sessions never
freeze, there's a human in the chair. And the honest sentence written into
the phase: cairn stays in the "refused at runtime" tier and says so. A
plugin cannot make unsafe paths unreachable by construction. This makes
refusals derivable and auditable, not airtight.

The same evening I wrote six of those axioms into my global Claude rules,
so the classification scheme follows me into every repo, not just this one.

### Worktrees that break on `.env`, `node_modules`, and port 3000

Gabriel Anhaia's post is the practical one and it is dead on. A fresh
worktree has the tracked files of one commit and nothing else. His fixes:
copy, never symlink, the ignored files the repo lists; install from the
lockfile with the repo's own package manager; hand each worktree a block of
ports written to a file git ignores.

Cairn already runs work, audit, and review in worktrees, and I'd been
getting away with it because my own repo happens to need almost none of
that setup. That's luck, not design. Two issues landed in **phase 30,
the-second-hardening**: setup is derived from the repo, never typed (the
lockfile decides the installer, `.worktreeinclude` decides what copies,
`.worktree-port` decides the port, and a repo with none of those gets no
setup line rather than an invented one), and git hooks join the protected
set as harness configuration, because a `post-checkout` hook that fires on
every worktree add is exactly what an agent should not be quietly editing.

Global rule number two: four steps every fresh worktree gets before anyone
touches code, ending with "run the baseline tests," so a red you caused is
distinguishable from a red you inherited.

### Blocking queues and channels

ThinkerQAQ's series isn't about AI at all, which is sort of the point.
Part nine frames every handoff between two execution units around three
questions: is one handoff atomic, can the receiver see everything the
sender wrote, and what ordering does the handoff actually promise. The trap
it names: a queue protects the message, not what the consumer does with
shared state afterward.

Cairn has handoffs that are plain files, a hook appends a JSONL line and
the server reads it later. I asked the three questions of each one and
found a read-trim-rewrite on a live log, which is a lost-update bug waiting
for a second session to arrive. Issue #351 in phase 30 makes the discipline
explicit: append-only logs rotate by rename and never rewrite the live
file, and temp files for atomic writes carry a pid plus a random suffix,
never a fixed name. Two other phases build on that, so #351 lands first.
Global rule number three: three questions, answered in a comment, at every
handoff.

### Claude Code and Codex in parallel on one repo

Arun's field report from Offrun covers one worktree per agent, the gotchas
he actually hit, and having the second agent review the first one's diff.
The one that got me: a reviewer silently misses every new file because
untracked files don't appear in `git diff` without `git add -N`. I grepped
cairn's review verb and the default target was what git tracks, not what
the author made.

That and four more mechanics became **phase 29, the-parallel-path**,
organized by one rule: per-worktree versus per-repository state is decided
by what the state is *about*, not where it's written. A session's task in
flight is per worktree; knowledge of the project (memory index,
observations, metrics, refusals) is per repository, keyed by the git common
dir. The ledger is append-only, so a union merge driver is correct by
construction. Stash `pop`, `apply`, and `drop` refuse when more than one
worktree exists, because stash is the destructive half. A rate-limited peer
seat is a skipped seat, recorded as such, never a failed run, and zero
seats left is said plainly as "no quorum." Every one of those was verified
against a specific line in the tree before it was filed.

### A UX bar for AI-generated design

Cairn has a design verb and a designer persona, and it had no acceptance
bar for whether the output was any good. zeeshani42's skill has one. The
easy move was to vendor it into cairn. I didn't, and the reason is a
number: cairn's resident context footprint was at 13,107 tokens against a
pinned budget of 13,145, a 38-token cushion, and the skill's description
alone is about 80.

**Phase 27, the-ux-seat** integrates by reference, never by vendoring. The
skill stays upstream under its MIT license, the user installs it, cairn
points at it when present and behaves identically when absent. A `ux`
review seat joins as the seventh default seat, gated on a new `touches-ui`
signal so server-only diffs never pay for it, and "render, don't describe"
becomes the floor the UI audit applies when a phase declares no criteria of
its own.

A sibling read that night, Dietrich Gebert's ponytail repo, added a `lean`
seat that asks whether code should exist at all. And one thing I explicitly
declined from it: always-on prompt injection at every turn, because it
competes with other plugins for the same hook points and costs resident
tokens forever. Writing down what you turned down is part of the loop too.

### The least autonomous architecture that works

The quietest article and the only one that produced merged code the same
week. h1manshu01's thesis: start with one call, add a workflow when the
process needs structure, reserve an agent for when the process itself has
to be discovered.

I went looking for places where cairn's markdown verbs ask the *model* to
execute a known sequence the server could just run. Found four. The claim
comment an agent posts when it starts an issue is now composed server-side
from a typed block. Every registered MCP tool carries annotations so the
harness knows what's read-only and what's destructive without asking. Review
dispatch defaults to automatic seat selection with the full panel as the
opt-in. And a sweep moved the remaining known sequences out of verb prose
and into code. Same law as the rest of phase 26: surface count flat,
footprint down or level. All four are on main.

## What I actually learned

Nobody has the whole picture of agentic coding yet. Not the labs, not the
people shipping harnesses, not me. What exists instead is a few thousand
engineers running the same experiment with slightly different variables
and writing up what broke, and if you read those write-ups the way you'd
read a bug report against your own assumptions, they're the best design
review you'll ever get, and they're free, and they're on the front page.

Six articles, six tree checks. Three said "the article is right about
you." Two named a gap I hadn't put words to. One confirmed my grain and
still shipped four fixes. Even the confirmation earned its keep, because
the decision now lives in a planning doc with a source instead of in my
head.

And the other half is that you have to put your own stuff out there,
because somebody is three hours into their own reading night right now,
and your post about the thing that broke is the one that fixes their tree.
Ideas in, ideas out, attribution on everything…

## The sources

Go read them. They're better than my summaries.

- Alex Aslam, [Why your agents need a constitution, not a prompt](https://dev.to/alex_aslam/why-your-agents-need-a-constitution-not-a-prompt-2181)
- Gabriel Anhaia, [Git Worktrees for Coding Agents Break on .env, node_modules and Port 3000. Fix All Three](https://dev.to/gabrielanhaia/git-worktrees-for-coding-agents-break-on-env-nodemodules-and-port-3000-fix-all-three-3781)
- ThinkerQAQ, Concurrency Programming (9): BlockingQueue and Channels, in the [Concurrency Programming series](https://dev.to/thinkerqaq)
- Arun (Offrun), [Running Claude Code and Codex in parallel on one repo: git worktree gotchas and cross-review](https://dev.to/offrun/running-claude-code-and-codex-in-parallel-on-one-repo-git-worktree-gotchas-and-cross-review-41eg)
- zeeshani42, I built a Claude skill that makes AI design like a senior UX designer ([author page](https://dev.to/zeeshani42); skill at [github.com/zeeshani42/your-ui-ux-designer](https://github.com/zeeshani42/your-ui-ux-designer), MIT)
- h1manshu01, [When to Use an AI Agent Instead of a Simple LLM Call](https://dev.to/h1manshu01/when-to-use-an-ai-agent-instead-of-a-simple-llm-call-djk)
- Dietrich Gebert, [ponytail](https://github.com/dietrichgebert/ponytail) (GitHub, MIT)

Cairn is MIT and the roadmap with phases 27 through 30 is public at
[github.com/eventually-consistent-code/cairn2](https://github.com/eventually-consistent-code/cairn2).
If one of your posts ends up in a phase's `## Source` section, I'll tell
you.
