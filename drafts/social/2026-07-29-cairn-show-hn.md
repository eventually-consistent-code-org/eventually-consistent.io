# Show HN draft (posting is John's click)

**Title:** Show HN: Cairn – project management for AI coding agents, mirrored to your real tracker

**URL:** https://eventually-consistent.io/projects/cairn/

**First comment (post immediately after submitting):**

Ever asked an AI agent to build something real — phases, a deadline, a boss who asks how it's going — and watched the plan evaporate when the context window rolled over? That's the problem cairn works on.

The core idea is tracker-first: every piece of work the agent plans or ships gets mirrored to the issue tracker your team actually looks at — Jira, GitHub, GitLab, Linear, Asana, Azure Boards, ClickUp, or a zero-credential local tracker that lives in the repo. Comments, worklogs, sprints, estimates, custom status vocabularies. The paper trail is the product; management visibility isn't an afterthought.

Three design choices I'd defend:

1. Verification is goal-backward, not checkbox-forward. A phase isn't done because the issues got closed; issues get closed because a verifier walked the goal backward and found evidence. Drift between plan and reality blocks the ship gate.

2. Memory has provenance. Lessons are cards pointing at the files and commits that prove them, with confidence grades that get re-checked at retro time. Stale memory outranking fresh reality is the failure mode we most wanted to avoid — we watched it happen in comparable tools.

3. Honest capability tables. The docs' comparison matrix cites the competing tools' own issue trackers for every claim (their sync gaps, their "not planned" closures), and our own gaps get the same treatment — the cost tracker is Claude Code-only and the docs say exactly why.

It started as a Claude Code plugin; as of this week one installer also wires it into Grok Build, Copilot CLI, Codex, Gemini CLI, Cursor (including a full port of the hook layer), OpenCode, and Zed. The server is plain MCP over stdio — 71 typed tools — and the verbs are markdown subroutines any capable agent can follow.

744 tests, 3 runtime dependencies, MIT. Repo: https://github.com/eventually-consistent-code/cairn2

Happy to answer anything — especially skeptical questions about whether agents closing their own tickets can ever be trustworthy. That skepticism is why the verification gate exists…
