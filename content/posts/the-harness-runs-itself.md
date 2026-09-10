+++
title = "The Harness Runs Itself: Budgeted, Headless, and Honest About Its Own Bugs"
date = 2026-09-10T00:00:00Z
draft = false
tags = ["cairn", "claude-code", "agentic-tooling", "headless", "automation", "ai", "meta"]
summary = "Four weeks after the work-harness post: cairn learned to keep its own documentation honest, run entire phases unattended inside a token budget, and — twice now — catch its own bugs in the act and ship the fixes. Here's what a self-running work harness actually looks like, receipts included."
+++

Last time I wrote about cairn, I asked what kind of harness you're
strapping around your AI — skeletal loop, discipline layer, or a full
work harness that holds the plan, the memory, and the verification. That
post ended with cairn planted firmly at the heavy end of the spectrum.

So here's the question this month answered: once the harness holds all
that state, can it *run itself*? Can you tell it "here are two phases,
here's a token budget, here's my push authorization — go," and come back
twenty minutes later to shipped code and an honest report?

Turns out yes. And the honest part is doing more work in that sentence
than the headless part…

## First, the docs had to stop lying

Before you let anything run unattended, you need to trust its
paperwork, and mine had been quietly rotting. Every release, the README's
tool counts drifted, the changelog missed phases, the architecture page
described a server five subsystems smaller than the real one. Classic
docs problem — nobody notices until an audit sweeps it, and my audit
swept fifteen findings in one pass, two of them bad enough to break a
new user's install.

So the first thing cairn grew was **documentation staleness as computed
math**. A deterministic check — no AI judgment anywhere in it — walks
every verified phase and asks two dumb questions: does the changelog
mention you, and has the documentation moved since your last recorded
work? Dumb questions are the good kind, because the answers are facts.
On its very first run it flagged four older phases whose entries had
silently never landed. Debt I didn't know I had, surfaced by arithmetic.

Generation got wired in behind the detection — verification *reports*
docs debt, shipping *generates* the catch-up entries behind the same
single confirmation you were already answering, and milestone completion
runs the full synthesis. The generated text can only land inside
explicitly marked sections, so the machine physically cannot clobber
hand-written prose. Not "is instructed not to" — cannot. An unmarked
section is an error, and identical rewrites are a proven no-op.

And the README now recomputes its own claims — verb counts, tool counts,
test totals, backend lists — straight from the live registry, failing CI
when they drift. It caught two real drifts in its first hour of
existence. The claims page and the comparison page work the same way
now. When your whole pitch is "claims that check out," the claims had
better maintain themselves…

## Then the harness learned to run on a budget

The headless build is one staging conversation, and everything about it
front-loads the parts that need a human. You pick the phases (or set a
budget and let the estimator pick them), you see honest cost ranges —
*ranges*, never point claims, calibrated from what past phases actually
cost — and you grant push authorization once, up front, scope-limited to
exactly the phases you approved.

That last piece deserves a beat. Back in August, a review council of
four rival AI vendors told me ship needed a human confirmation before
anything pushes, and I took that recommendation over cairn's own
no-action vote. Headless mode doesn't delete that gate — it *moves* it.
The confirmation happens at run start, gets recorded with a timestamp in
the run's manifest, and the manifest can never widen its own authority.
Decline it and the run still executes headless, just nothing pushes.
The gate survived; only its position moved.

Then you walk away. Each phase plans if it needs to, fans work out to
parallel agents in isolated checkouts, verifies goal-backward, ships
under the pre-authorization, and narrates every transition on a public
tracker issue while a spend ledger checks the budget at every boundary.
The guarantee is stated the same way everywhere: no new work starts past
the ceiling, in-flight work finishes, overshoot is bounded by at most
one wave — recorded, never hidden. And when you come back, a report is
waiting: what shipped, what it cost against the estimate, every
unattended decision with the principle that resolved it, and why the run
ended.

## Then it had to prove it — and this is my favorite part

A month ago I'd have told you the machinery worked because the tests
passed. The milestone I just closed was named "prove it" for a reason:
an unexercised headless executor is exactly the credibility hole a
skeptical adopter finds first, so the acceptance test was a real run
with real cargo.

**Run one** took seven bugfixes across two phases — the harness's own
friction backlog — and shipped all of them, unattended, verified, pushed
under the staged authorization, in about twenty-two minutes against a
thirteen-hour human estimate. The tracker narrated the whole thing live.
And the budget meter read *zero the entire run*.

Not because the run was free. Because the ledger couldn't see: the
metrics pipeline never received the worker agents' spend, so the ceiling
enforced faithfully against a number that was blind. The machinery being
tested caught its own instrumentation gap, headlined it in its own
report, and a reconstruction from the agents' records showed the run
would have played out identically with working instrumentation —
finishing about twelve percent over, squarely inside the documented
one-wave overshoot bound. The contract held; the plumbing owed. The fix
shipped in the very next release, proven by a live refusal test.

**Run two** closed the milestone with the fixed ledger, and the
estimator — now calibrated on run one's actuals — staged a range forty
percent narrower and landed the real cost dead in the middle of it,
about thirteen dollars for a four-item phase. Then the boundary check
fired its first live STOP verdict… on an accounting artifact. The ledger
counts cached-context traffic in its token unit; the estimator that set
the ceiling deliberately doesn't. The dollar axis stayed perfectly sane,
which is what diagnosed it in seconds. Filed, contract honored, run
completed.

Run one found the ledger blind. Run two found its glasses were the
wrong prescription. Each run's machinery caught its own next bug, and
both of those catches are public — the first run is now a full case
study sitting next to the council piece, blind ceiling and all. I keep
saying the differentiator is honesty as mechanism rather than marketing,
and I can't think of a better demonstration than software that files
tracker issues against itself mid-run…

## And then it ate its own dog food, again

The same milestone wired cairn's docs connector to cairn's own repo —
seventy-nine pages published into a Docusaurus site, every page stamped
with the release it came from, idempotent on re-publish. The very first
self-publish caught two real adapter bugs no test had found: false
orphan warnings on case-insensitive filesystems, and container pages
silently dropping their images. Both fixed with regression tests before
the run ended. There's a lesson in there that never stops being true:
nothing audits your product like being your own most demanding user.

Where the numbers stand today: 39 verbs, 84 typed MCP tools, 1,260
passing tests, 8 tracker backends, version 2.4.0 live on npm — and
every one of those numbers in the README computed itself.

## What this proves, and what it doesn't

One live refusal on record beats any promise about ceilings — even
though it fired for the wrong reason, that's precisely what field tests
are for. The overshoot guarantee is bounded, not a hard wall; agent
spend is only cleanly observable at boundaries, and pretending otherwise
would just be a tighter-sounding lie. The resume machinery is armed but
has never survived a real kill. And two runs is two runs — a pattern,
not a law.

But the shape of the thing is now proven end to end: research feeds
plans, plans feed budgeted unattended execution, execution feeds
verification and memory, memory recalibrates the next estimate, and
every step leaves a paper trail a non-engineer could audit. That's what
I meant by a work harness. It just runs itself now, inside a boundary
you set, with receipts…

## Try it

```
/plugin marketplace add eventually-consistent-code/cairn2
/plugin install cairn
```

or `npm i @eventually-consistent/cairn-server` for any bare-MCP harness.
Start with
[the quickstart](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/00-quickstart.md),
then read the two case studies back to back —
[the council review](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-council.md)
and
[the batch run](https://github.com/eventually-consistent-code/cairn2/blob/main/docs/case-study-batch-run.md).
One is four rivals judging my software; the other is my software judging
itself. It's harder on itself than they were!

-John Reed
