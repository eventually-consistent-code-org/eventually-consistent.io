+++
title = "TagManager"
date = 2026-08-03T00:00:00Z
draft = false
tags = ["python", "multi-cloud", "aws", "finops", "cloud-governance"]
summary = "Multi-cloud tag compliance and storage cost analysis in one container — scan AWS, Azure, and GCP, catch the tags that are missing or wrong, and see what stale data is costing you."
repo = "https://github.com/eventually-consistent-code/TagManager"
+++

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
guessing. Every number traces back to something it measured, which matters when
you're about to tell finance you can cut a storage bill or about to delete data
someone might still need. Explainable beats magic when real money and real data
are on the line.

Who's it for? Cloud and platform teams drowning in tribal knowledge — the folks
who inherited three clouds' worth of resources nobody documented, need tag
governance that's visible instead of folklore, and would also like to stop
paying to store data that hasn't been touched since 2022. Run `docker compose
up`, point your team at the dashboard, and make the drift visible. Built with
Nick Bitzer.

The code is public: [github.com/eventually-consistent-code/TagManager](https://github.com/eventually-consistent-code/TagManager).
