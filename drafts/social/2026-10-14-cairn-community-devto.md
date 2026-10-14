---
title: "Six dev.to Articles Walked In, Four Roadmap Phases Walked Out: Why the Community Is the Best Design Review Your Project Will Ever Get"
published: false
description: "One evening of reading other people's posts about agentic coding turned into three new global rules on my machine, four new phases on cairn's roadmap, and four issues already shipped. Here's the loop that got me there, who to thank, and why I think you should run it too."
tags: ai, opensource, devtools, community
canonical_url: https://www.eventually-consistent.io/posts/the-community-is-the-design-review/
cover_image: https://www.eventually-consistent.io/img/cairn-logo-h.jpg
---

<!-- dev.to draft. Posting is John's click. Two source URLs could not be
     resolved by search (ThinkerQAQ part 9, zeeshani42's post) — author
     pages are linked instead; swap in the exact article URLs before posting. -->

Ever have one of those evenings where you sit down to "catch up on reading"
and three hours later your roadmap has four new phases on it?

That was me on October 13th. I'd been heads-down on cairn, the work harness
I've been building for AI coding agents, for a couple of months straight,
shipping release after release and mostly talking to my own test suite. I
opened dev.to to decompress. Six articles later I had rewritten how I think
about guards, how I set up a worktree, how I write to a log file, and how a
review seat gets picked, and every one of those changes traces back to
somebody else's post that I did not write and could not have written,
because I was too close to my own code to see it…

So this article isn't about cairn's features. It's about the loop that
turned six strangers' ideas into real work, because I think that loop is the
most underrated thing about building in the open right now, and I want to
show you exactly how it ran.

## The loop, in one paragraph

Read the article whole. Then, before you form an opinion, go check your
own tree for the thing it describes. Not "does this sound right," but "does
my code actually do this, at which line?" If the article is right about
you, file the fix as a phase with the article named as its source. If the
article is wrong about you, or your grain already matches, write that down
too, because a confirmed decision is worth almost as much as a new one. Then
pay it forward: attribute every file that borrows the idea, paraphrase the
reasoning, copy no blocks. That's it. Clean-room reading with receipts.

Here's what fell out of that loop, article by article.

## 1. "Why your agents need a constitution, not a prompt" (Alex Aslam)

This one hurt, in the good way. Alex's argument is that rules written per
incident never converge: each new prohibition leaves the adjacent behavior
uncovered, so you end up on a treadmill, forever one failure behind. What
you need instead is a classification scheme that makes the *next* unseen
case recognizable.

I went and looked at cairn's run guard. Its header listed seventeen issue
numbers. Seventeen separate times something went wrong, and seventeen
separate times I added a rule justified by the issue that caused it. The
article's critique was accurate about my code on exactly that point, so
it became **phase 28, the-constitution**: five to seven axioms in the
architecture doc that classify what counts as what (the user's data versus
scratch, a destructive action, a directive versus data, done versus
claimed, the owner's checkout, the harness's own configuration). Every
guard and every ADR names the clause it derives from, and a rule that
derives from no clause is itself the finding. The run guard gets rewritten
to classify instead of enumerate, with all seventeen existing tests kept
green as the regression corpus.

Two mechanisms rode along: the server can now re-run the owner-declared
`verify:` line itself at close time and report observed-versus-claimed,
and headless runs get a circuit breaker that can freeze a run that keeps
getting refused. Interactive sessions never freeze, there's a human in the
chair for those. And the honest part, written into the phase: cairn stays
in the "refused at runtime" tier and says so. A plugin cannot make unsafe
paths unreachable by construction. This phase makes the refusals derivable
and auditable, not airtight…

The same evening I wrote a six-clause version of those axioms into my
global Claude rules, so every project I touch gets the classification
scheme, not just cairn.

## 2. "Git Worktrees for Coding Agents Break on .env, node_modules and Port 3000" (Gabriel Anhaia)

Gabriel's post is the practical one, and it's dead on. A fresh worktree has
the tracked files of one commit and nothing else: no `.env`, no
`node_modules`, and a dev server that reaches for the same port every other
worktree wants. His fixes are specific and correct: copy, never symlink,
the ignored files the repo lists; install from the lockfile with the repo's
own package manager; give each worktree a block of ports written to a file
git ignores.

Cairn already uses worktrees for work, audit, and review, and I had been
getting away with it because my own repo happens to need almost none of
that. That's luck, not design. So two issues landed in **phase 30,
the-second-hardening**: setup is derived from the repo, never typed (the
lockfile decides the installer, `.worktreeinclude` decides what copies,
`.worktree-port` decides the port, and a repo with none of those gets no
setup line rather than an invented one), and git hooks join the protected
set as harness configuration, because a `post-checkout` hook that runs on
every worktree add is exactly the kind of thing an agent should not be
quietly editing.

This one also became a global rule on my machine, four steps every fresh
worktree gets before anyone touches code. Step four is "run the baseline
tests," so a red you caused is distinguishable from a red you inherited.

## 3. "Concurrency Programming (9): BlockingQueue and Channels" (ThinkerQAQ)

Not an AI article at all, and that's sort of the point. ThinkerQAQ's series
is about the fundamentals, and part nine frames every handoff between two
execution units around three questions: is one handoff atomic, can the
receiver see everything the sender wrote, and what ordering does the
handoff actually promise. The trap it names is that a queue protects the
*message*, not what the consumer does with shared state afterward.

Cairn has a bunch of handoffs that are plain files: a hook appends a JSONL
line, the server reads it later. I asked the three questions of each one
and found a read-trim-rewrite on a live log, which is a lost-update bug
waiting for a second session to show up. So issue #351 in phase 30 makes
the discipline explicit: append-only logs rotate by rename and never
rewrite the live file, and temp files for atomic writes carry a pid plus a
random suffix, never a fixed `.tmp`. Two other phases build on that
discipline, so #351 lands first. And yes, a third global rule: three
questions, answered in a comment, at every handoff.

## 4. "Running Claude Code and Codex in parallel on one repo" (Arun, Offrun)

Arun's post is a field report: one worktree per agent, the gotchas he
actually hit, and how to have the second agent review the first one's diff.
The gotcha that got me was the reviewer silently missing every new file
because untracked files don't show up in `git diff` without `git add -N`.
I grepped cairn's review verb and, sure enough, the default review target
was what git tracks, not what the author made.

That plus four more mechanics became **phase 29, the-parallel-path**. The
rule that organizes it: per-worktree versus per-repository state is decided
by what the state is *about*, not where it's written. A session's task in
flight is per worktree; knowledge of the project (memory index,
observations, metrics, refusals) is per repository and keys by the git
common dir. The ledger is append-only, so a union merge driver is correct
by construction. Stash `pop`, `apply`, and `drop` refuse when more than one
worktree exists, because stash is the destructive half. And a rate-limited
peer seat is a skipped seat, recorded as such, never a failed run, with
zero seats left said plainly as "no quorum." Every one of those I verified
against a specific line in my tree before filing.

## 5. "I built a Claude skill that makes AI design like a senior UX designer" (zeeshani42)

Cairn has a design verb and a designer persona, and it had no acceptance
bar for whether the output was any good. zeeshani42's skill has one. The
easy move would have been to vendor the skill into cairn. I didn't, and
the reason is a number: cairn's resident context footprint was sitting at
13,107 tokens against a pinned budget of 13,145, a 38-token cushion, and
the skill's description alone is about 80.

So **phase 27, the-ux-seat** integrates by reference, never by vendoring.
The skill stays upstream under its own MIT license, the user installs it,
and cairn points at it when present and behaves identically when absent.
A `ux` review seat becomes the seventh default seat, gated on a new
`touches-ui` signal so server-only diffs never pay for it, and "render,
don't describe" becomes the floor the UI audit applies when a phase
declares no criteria of its own. Attribution in every file that borrows
the bar, paraphrase the criteria, copy no blocks…

A sibling read the same night, Dietrich Gebert's ponytail repo on GitHub,
contributed a `lean` seat that asks whether code should exist at all, with
ponytail's never-cut list as the floor. And one thing I explicitly did
*not* adopt from it: always-on prompt injection at every turn, because it
competes with other plugins for the same hook points and adds resident
tokens every turn. Writing down what you declined is part of the loop.

## 6. "When to Use an AI Agent Instead of a Simple LLM Call" (h1manshu01)

The quietest article and maybe the most consequential, because it was the
only one that produced shipped code the same week. The thesis: pick the
least autonomous architecture that reliably solves the problem. Start with
one call, add a workflow when the process needs structure, reserve an agent
for when the process itself has to be discovered.

I read that and went looking for places where cairn's markdown verbs ask
the *model* to execute a known sequence that the server could just do.
Found four. The claim comment an agent posts when it starts an issue is now
composed server-side from a typed block instead of being prose the model
assembles. Every registered MCP tool carries annotations so the harness
knows what's read-only and what's destructive without asking. The review
dispatch defaults to automatic seat selection, with the full panel as the
opt-in. And a sweep moved the remaining known sequences out of verb prose
and into code. Four issues, same law as the rest of phase 26: surface count
flat, footprint down or level. All four are merged.

## What I actually learned

Nobody has the whole picture of agentic coding yet. Not Anthropic, not the
people shipping harnesses, not me. What exists instead is a few thousand
engineers running the same experiment with slightly different variables and
writing up what broke, and if you read those write-ups the way you'd read a
bug report against your own assumptions, they are the best design review
you will ever get, and it's free, and it's waiting on the front page.

The thing that makes it work is the grounding step. An idea you admire is
worth nothing to your project until you've checked it against your tree
and found the line where you're wrong. Six articles, six tree checks, and
the scorecard was: three "the article is right about me" (constitution,
worktrees, concurrency), two "right about a gap I hadn't named" (parallel
path, UX bar), one "already my grain, now with four fewer places the model
has to improvise" (least autonomous). Even the confirmations earned their
keep, because now the decision is written down with a source instead of
living in my head.

And the other half of the loop is that you have to put your own stuff out
there, because somebody else is three hours into their reading night right
now, and your post about the thing that broke is the one that fixes their
tree. That's the whole economy. Ideas in, ideas out, attribution on
everything…

## The sources

Go read them. They're better than my summaries.

- Alex Aslam, [Why your agents need a constitution, not a prompt](https://dev.to/alex_aslam/why-your-agents-need-a-constitution-not-a-prompt-2181)
- Gabriel Anhaia, [Git Worktrees for Coding Agents Break on .env, node_modules and Port 3000. Fix All Three](https://dev.to/gabrielanhaia/git-worktrees-for-coding-agents-break-on-env-nodemodules-and-port-3000-fix-all-three-3781)
- ThinkerQAQ, Concurrency Programming (9): BlockingQueue and Channels, part of the [Concurrency Programming series](https://dev.to/thinkerqaq)
- Arun (Offrun), [Running Claude Code and Codex in parallel on one repo: git worktree gotchas and cross-review](https://dev.to/offrun/running-claude-code-and-codex-in-parallel-on-one-repo-git-worktree-gotchas-and-cross-review-41eg)
- zeeshani42, I built a Claude skill that makes AI design like a senior UX designer ([author page](https://dev.to/zeeshani42), skill at [github.com/zeeshani42/your-ui-ux-designer](https://github.com/zeeshani42/your-ui-ux-designer), MIT)
- h1manshu01, [When to Use an AI Agent Instead of a Simple LLM Call](https://dev.to/h1manshu01/when-to-use-an-ai-agent-instead-of-a-simple-llm-call-djk)
- Dietrich Gebert, [ponytail](https://github.com/dietrichgebert/ponytail) (GitHub, MIT)

Cairn is open source (MIT) and the roadmap with phases 27 through 30 is
public: [github.com/eventually-consistent-code/cairn2](https://github.com/eventually-consistent-code/cairn2).
If one of your posts ends up in a phase's `## Source` section, I'll tell
you.

-John Reed
