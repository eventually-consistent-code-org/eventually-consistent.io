+++
title = "OKRDoKey"
date = 2026-08-03T00:00:00Z
draft = false
tags = [ "typescript", "self-hosted", "okr-tracking", "ai", "docker" ]
summary = "A simple, self-hostable OKR tracker for small teams — spreadsheet-simple, one container, your data, with optional AI that coaches your key results instead of writing them for you."
repo = "https://github.com/eventually-consistent-code/okrdokey"
+++

OKRDoKey started with a conversation. A friend and old colleague and I were catching up the other day, and he got onto how much of a pain OKRs and KPIs are to actually run — not the idea, the tooling. Every big platform out there is overkill: dashboards stacked on dashboards, endless config, and somewhere under all of it the one thing OKRs were meant to do — keep a team pointed at a few measurable outcomes — gets buried. So they're miserable to manage, people quietly stop updating them, and the whole practice rots. I figured I'd try to solve that.

So here's what it does. You write objectives, hang measurable key results off them (numeric, percent-complete, or done/not-done — and yeah, decreasing-is-good works too), and the scores roll up on their own: key-result progress into an objective score into a cycle dashboard. Every check-in carries a red/yellow/green confidence read, and whether an objective is on-track, at-risk, or behind gets computed from progress against time elapsed — never hand-picked to make the slide look better. Quarterly cycles fill in their own dates, and when the quarter ends one click closes it out and carries the unfinished work forward with fresh baselines. Trends and history rebuild themselves from your check-ins, so you get score-over-time charts without any extra data entry.

The stuff I'm proudest of is where it stays honest. You can push key-result values straight from a script or CI with an API token, or wire a KR to a GitHub milestone or a Jira query and let the sync loop move the number as work actually closes. KPIs live right next to OKRs — steady-state stuff like uptime or MRR gets its own lane with target bands, so your "keep it green" metrics don't sneak in and masquerade as objectives. And there's a guided wizard with a pile of templates that quietly keeps "launch the thing" tasks from ending up as key results, which is the single most common way OKRs go bad.

There's an AI angle too, but on purpose it's a small one. Bring your own Anthropic key — per team or instance-wide — and the wizard can draft a couple of measurable KR suggestions from an objective, or critique one you typed yourself. It's a coach, not an autopilot. Suggestions only prefill the form, every baseline it invents is a placeholder you replace with your real number, keys never touch the browser, prompt bodies never get logged, and if you don't want any of it you flip one env var and every AI route and button disappears. The whole thing runs fine with the AI switched off.

The other half of the point is owning it. OKRDoKey is one Docker container and one SQLite file — no Redis, no Postgres, no external services, backup is literally copying a file. It's a TypeScript monorepo under the hood (Fastify + Drizzle + SQLite on the API, a React SPA on the front, and a shared set of Zod schemas that drive validation, the OpenAPI docs, and the web client all from one source of truth). SSO works with any OIDC provider and it's not gated behind an "enterprise" tier, because that always felt like a scam to me.

Who's it for? Small teams — startups, side ventures, an engineering group inside a bigger company, anybody homelabbing their own stack — who want the discipline of OKRs without handing their strategy over to somebody else's SaaS. If you can run `docker compose up`, you can run this.

And the name? That's the whole idea — when the board finally goes all green, it's OKRDoKey.

The code is public: [github.com/eventually-consistent-code/okrdokey](https://github.com/eventually-consistent-code/okrdokey).
