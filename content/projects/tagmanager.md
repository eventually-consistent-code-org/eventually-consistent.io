+++
title = "TagManager"
date = 2026-08-03T00:00:00Z
draft = false
tags = ["python", "multi-cloud", "aws", "finops", "cloud-governance"]
summary = "Multi-cloud tag compliance and storage cost analysis in one container — scan AWS, Azure, and GCP, catch the tags that are missing or wrong, and see what stale data is costing you."
repo = "https://github.com/eventually-consistent-code/TagManager"
images = ["img/tagmanager-icon.jpg"]   # square mark — social-share / og:image
+++

<p style="text-align:center; margin:0 0 2rem;">
  <img src="/img/tagmanager-logo.jpg" width="300"
       style="max-width:80%; height:auto;"
       alt="TagManager logo — a glass amber app tile holding a luggage-style price tag, beside the wordmark TagManager">
</p>

It's 2am, something's on fire, and the one EC2 instance at the center of it has
no owner tag — or worse, a wrong one that sends you chasing the wrong team.
That's the whole reason TagManager exists. Cloud tags are how you answer "what is this, who owns it, what
environment is it in" — but tags only do that if they're actually there and
actually consistent, and in a real account they drift the second you look away…

So TagManager scans your cloud accounts on a schedule, compares every
resource's tags against your canonical rules — allowed keys, allowed values —
and surfaces every deviation so you can fix drift before it bites you. It reads
AWS through the Resource Groups Tagging API, Azure through Resource Graph, and
GCP through Cloud Asset Inventory, and normalizes all three into one catalog so
your rules don't care which cloud a resource lives on. GCP labels just become
tags like everything else. One container gives you a read-only web dashboard, a
JSON API, an in-process scheduler that sweeps every account hourly, and OIDC
auth that fails *closed* if you misconfigure it, not open.

It started life as a plain AWS EC2 tag checker — a CLI I ran to catch
untagged instances and fail a CI build when tags were out of policy. That
original tool still ships and still works; it grew a platform around it instead
of getting replaced. Along the way it picked up a second job that surprised me
how useful it is: storage age and cost analysis. Point it at an S3 bucket (or
Azure Blob, GCS, or a local/SMB path) and it rolls objects into age bands by
last-modified, prices what the stale data costs you every month, and projects
the savings of each option — delete, age-out rules, intelligent tiering,
archive — with break-even months and honest small-object math built in. Then it
generates the actual lifecycle configs and move manifests so you can go do
something about it.

Here's the part I care about most: there's no black box. The recommendations
are signal-driven and deterministic — thresholds, request rates, access
patterns from CloudTrail and S3 access logs, data-type awareness — not a model
guessing. Every one carries a confidence label (high / medium / low) and names
the evidence behind it, so a recommendation backed by real request-rate
telemetry doesn't read the same as one inferred from age alone. Every number
traces back to something it measured, which matters when you're about to tell
finance you can cut a storage bill or about to delete data someone might still
need. Explainable beats magic when real money and real data are on the line.

The newest piece is the dry-run diff, and it's the one I'd want most if I were
running this against a bucket that matters. Before anything gets applied, it
reads your bucket's live lifecycle and tiering config and shows you, rule by
rule, what an apply *would* do: what gets added, what changes, what's already
fine — and, loudly, what would be removed. That last one is the trap. S3
replaces a bucket's entire rule set when you apply a lifecycle config, so any
live rule you didn't generate gets silently dropped. TagManager tells you which
ones instead of letting you find out later. It writes nothing at all — the read
path is guarded by a test that asserts it never calls a mutating API — and if
it can't read a config, it says "no permission," never "no config." Can't-see
and nothing-there are not the same answer, and a tool that conflates them will
eventually lie to you at the worst possible moment…

All of it runs the same two ways: `docker compose up` for the web app —
configure scan targets, launch scans, watch them run, read the cost and savings
pages, download generated artifacts as a zip — or the CLI, shipping in the same
image, doing the same work with no web server involved. 281 tests green behind
it.

Who's it for? Cloud and platform teams drowning in tribal knowledge — the folks
who inherited three clouds' worth of resources nobody documented, need tag
governance that's visible instead of folklore, and would also like to stop
paying to store data that hasn't been touched since 2022. Run `docker compose
up`, point your team at the dashboard, and make the drift visible.

The code is public: [github.com/eventually-consistent-code/TagManager](https://github.com/eventually-consistent-code/TagManager).

I first wrote this script with a colleague, Nick Bitzer, who showed me the ropes
on Python and has been a trusted friend and advisor ever since. Check him out on
[LinkedIn](https://www.linkedin.com/in/nick-bitzer) and
[GitHub](https://github.com/nbitzer/nbitzer).
