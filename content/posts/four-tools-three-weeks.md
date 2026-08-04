+++
title = "Four Tools in Three Weeks: What Building With AI Actually Looked Like"
date = 2026-08-04T00:00:00Z
draft = false
tags = ["ai", "claude-code", "agentic-tooling", "cairn", "tooling", "workflow", "meta"]
flashNew = true   # home-list gold "New!" badge + pulse; cleared a week after publish
summary = "Four real products — a multi-cloud tag and storage tool, a resume tailor, an OKR tracker, and the planning system underneath all of it — built and shipped in about three weeks. What made it fast, what I got wrong, and what AI still can't do for you."
+++

Here's a question I couldn't have answered honestly a year ago: how long does it
take one person to build a usable product?

Not a demo. Not a weekend toy that dies the moment someone else runs it. A real
thing — with auth that fails closed, a test suite, a container that boots on
someone else's machine, docs, and a URL you can hand to a stranger.

My honest answer used to be *months*. Plural. And that was for one of them, if
the evenings held out and I didn't lose the thread somewhere around week five.

Over the last three weeks I shipped four…

## The receipts

I'm not going to hand-wave this, because the whole point is the numbers. All of
this is in the git history and in the planning artifacts, which is exactly why I
can quote it back to you:

| Project | Built | Days | Commits | Where it landed |
|---|---|---|---|---|
| **cairn** | Jul 12 – Jul 31 | 20 | 442 | 744 tests, 65 server tools, 8 harnesses |
| **OKRDoKey** | Jul 30 – Jul 31 | 2 | 112 | 3 milestones, 272 tests + 4 e2e |
| **TagManager** | Jul 29 – Aug 4 | 7 | 129 | 4 milestones, 281 tests |
| **tailor** | Jul 29 – Aug 4 | 7 | 66 | Live on the internet, taking payments |

Roughly 750 commits in twenty-four days. Four products, three of which I'd
never written a line of before this month.

The TagManager row deserves a footnote, because it's my favorite part of this
whole story. Its first commit is dated **December 6, 2016**. It was a Python 2
script I wrote with a colleague to catch untagged EC2 instances, and then it sat
untouched for nearly ten years. I picked it back up on July 29th. In seven days
it became a multi-cloud compliance platform with a web dashboard, a storage cost
engine, and a dry-run diff — a thing I had genuinely intended to "get back to
someday" since the Obama administration…

That's the part I want you to sit with. It wasn't that I lacked the ability for
ten years. I lacked the *runway*. The gap between "I know exactly what this
should be" and "it exists" was always measured in months I didn't have.

## What I actually built

### tailor — [try it right now](https://tailor.eventually-consistent.io)

You can spot an AI-written resume in about four words. Usually "results-driven
professional." Same buzzwords, same faint smell of a chatbot that never met you,
and every recruiter on earth has learned to skim past it.

tailor is the antidote. You paste in a job description, and it hands back a
resume and cover letter tuned to that specific job — grounded in *your* real
experience, written in *your* voice. The grounding is the whole product: it
works from a master record of your actual career, and it will not invent a
credential, a date, or a job you didn't have. Ever. That was non-negotiable from
the first commit, and it's enforced in the code, not in a prompt asking nicely.

It's live, it's multi-user, and **your first resume and cover letter are
free** — go run it against a job you actually want and see what comes back. If
you like it, a $5 pack gets you ten more, or $9/month gets you fifty. That's it.
No annual contract, no "book a demo," no seat minimum.

If you're job hunting right now, this is the one to try. Twenty minutes of
tailoring per application is the tax everyone pays and nobody talks about —
tailor makes it about ninety seconds, and the output still sounds like you.

→ [tailor.eventually-consistent.io](https://tailor.eventually-consistent.io) ·
[project page](/projects/tailor/) ·
[GitHub](https://github.com/eventually-consistent-code/tailor)

### TagManager — the ten-year-old script that grew up

Multi-cloud tag compliance and storage cost analysis in one container. It scans
AWS, Azure, and GCP, normalizes all three into one catalog, and checks every
resource against your canonical tagging rules so drift is visible instead of
tribal knowledge. Then it does a second job I didn't plan on: points at your
buckets, rolls objects into age bands, prices what stale data costs you monthly,
and projects the savings of each fix with break-even math and honest
small-object accounting.

The newest piece is a dry-run diff — it reads your bucket's live lifecycle
config and shows rule by rule what an apply *would* change, including the rules
that would get silently dropped. Writes nothing.

→ [project page](/projects/tagmanager/) ·
[GitHub](https://github.com/eventually-consistent-code/TagManager)

### OKRDoKey — spreadsheet-simple OKRs

This one started as a conversation. A friend and old colleague was venting about
how miserable OKR tooling is — not the practice, the tools. Dashboards stacked
on dashboards, and somewhere under it all the one thing OKRs were meant to do
gets buried, so people quietly stop updating them and the whole practice rots.

So: objectives, measurable key results, check-ins with a confidence read, scores
that roll up on their own. On-track vs. at-risk gets computed from progress
against time elapsed — never hand-picked to make the slide look better. One
container, your data, optional AI that coaches your key results instead of
writing them for you.

When the board goes all green, it's OKRDoKey.

→ [project page](/projects/okrdokey/) ·
[GitHub](https://github.com/eventually-consistent-code/okrdokey)

### cairn — the reason the other three exist

cairn is the planning system the rest of this was built with. Plans live in git,
every piece of work mirrors to your real tracker (Jira, Linear, GitHub,
ClickUp, Azure, or local), and a verification gate refuses to call anything done
without evidence. It runs across Claude Code, Cursor, Codex, Gemini CLI, Copilot
CLI, Zed, OpenCode, and Grok.

Everything in this post — every number, every decision I quote — came out of
cairn's artifacts. That's not a coincidence. It's the argument.

→ [project page](/projects/cairn/) ·
[GitHub](https://github.com/eventually-consistent-code/cairn2)

## The decisions that actually made it fast

Speed didn't come from typing faster. It came from a handful of choices that
each killed an entire category of rework. These are the ones that paid:

**Catch spec bugs before code exists.** Twice on TagManager, the plan-checker
pass blocked a real defect while the phase was still a document. One draft
priced tiered storage per rollup cell — restarting the 50 TB pricing ladder for
every cell, which would have overstated savings on every bucket it touched.
Fixing that in a plan cost ten minutes. Fixing it after it shipped into a
finance conversation costs your credibility.

**Verify adversarially, not agreeably.** On OKRDoKey's AI phase, 194 tests were
green and lint was clean. Then a verification pass explicitly prompted to
*refute* the work found three real defects — a retry contract that didn't cover
the throw case, a schema mismatch that turned a model quirk into a 500, and
phantom test coverage: a mock mode declared, implemented, and never once
assigned in any test. Green tests were lying. On TagManager the same approach
outright refuted a phase for omitting Glacier's per-object overhead.

That gave me a rule I now apply everywhere: *a retry test that only asserts the
final status passes identically with the retry deleted.* Count the invocations
or you've tested nothing.

**Smoke the real container, not just the test suite.** Two of OKRDoKey's three
worst bugs were invisible to both vitest and Playwright, because they only exist
in a production build: Secure cookies breaking plain-HTTP self-hosts (login
returns 200, the browser never sends the cookie back, every next request 401s),
and a native module needing a newer glibc than the base image had. Unit tests
cannot see either one. A `docker run` and a `curl` see both in thirty seconds.

**Fail closed.** TagManager refuses to serve on an unrecognized auth mode rather
than falling through to open access. One line of posture that means a typo in a
config file can never quietly publish your cloud inventory to the internet.

**Explainable over magic.** TagManager's recommendations are deterministic and
signal-driven — thresholds, request rates, access logs — and each one names its
own confidence and evidence. No model guessing. When you're about to tell
finance you can cut a bill, or delete data someone might still need, "the AI
said so" is not an answer.

## What I changed later

Nothing survived contact unchanged. The interesting part is that the changes
were cheap:

- **TagManager got rescoped mid-milestone.** Halfway through the signals work I
  decided the whole thing should be a containerized web app first, with the CLI
  preserved as a first-class equal — so the plan got reordered: web app, then
  sharpen what it knows. Reordering a roadmap took an afternoon because the
  roadmap was a real artifact and not a vibe.
- **tailor went from a local single-user script to a multi-user SaaS on AWS** —
  auth, billing, a real deploy — inside the same week it was born.
- **cairn's whole milestone flipped.** It started as a marketing launch and
  turned into a multi-harness port plus an npm release, because once it worked
  in one place the obvious question was why it only worked in one place.
- **And I broke my own process once, which is the useful one.** During
  TagManager's third milestone I ran the work through GitHub PRs directly and
  let the planning artifacts go hollow — empty plans, drifted status, a roadmap
  claiming things the repo didn't reflect. It didn't hurt on day one. It hurt on
  day four when I couldn't answer what was actually done. That drift is exactly
  what the verification gate exists to catch, and I'd routed around it.

## The honest part

I don't want this to read as "AI wrote four products." It didn't, and anyone
selling you that is selling you something.

Look at what I actually spent my hard hours on: a native module that needed
glibc 2.38 against a base image that had 2.36. Jira deleting a REST endpoint out
from under an adapter. `npm test | grep` swallowing an exit code so a red suite
reported green. A rate limiter whose `groupId` doesn't merge counters the way
the docs imply. Research that confidently reported a library version that
doesn't exist — which is the failure mode you have to stay awake for, because it
arrives sounding certain.

That's the job. It's still the job. What changed is the ratio. The mechanical
distance between a clear idea and a working artifact collapsed, so nearly all my
time went to the decisions that actually needed a human: what fails closed, what
gets refuted before it ships, what number I'd be willing to defend to finance.

Force multiplier is a phrase I've heard in every strategy deck for a decade and
never once believed. I believe it now. Not because the machine thinks for me —
because it removed the months of grunt work that used to sit between me and the
part I'm actually good at.

## What I got back

The thing I did not expect was how it felt.

I haven't been this creative in years. Somewhere in the last decade building
turned into a thing I *planned* to do — a someday, a backlog, a repo I'd get
back to. Three weeks ago it turned back into a thing I just *did*, and the
difference in my head is enormous. I'd forgotten that shipping is its own fuel:
you finish one, and the next idea shows up before you've closed the tab.

Ten years that tag script sat there. Seven days to finish it.

So go break something of mine. [tailor](https://tailor.eventually-consistent.io)
is live and your first one's free — that's the one I'd start with. The rest are
`docker compose up` and yours to keep. All four repos are public, the planning
artifacts are in every one of them, and if you find something wrong I genuinely
want the issue.

More to come…
