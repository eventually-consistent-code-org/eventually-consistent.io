# r/ClaudeAI draft (posting is John's click)

**Title:** I built a Claude Code plugin that mirrors everything the agent does to your real issue tracker (Jira/GitHub/Linear/+5) — and refuses to call work "done" without evidence

**Body:**

Like a lot of you, I kept hitting the same wall: Claude Code is brilliant inside a session and amnesiac across them. Plans lived in chat scrollback. My actual Jira board never heard about any of it. And "done" meant "the agent said done."

So I built cairn. The pitch in one line: give the agent what a real project needs — a plan that lives in git, work items in your real tracker, memory that survives the session, and a verification gate with trust issues.

What it does:

- **37 verbs** (`/cairn:new`, `plan`, `work`, `verify`, `ship`, `trace`, `retro`, …) over **71 MCP tools**
- **8 tracker backends** — GitHub, GitLab, Jira, Asana, Azure Boards, ClickUp, Linear, or a zero-credential local tracker committed to your repo. Two-way: PM edits in the tracker flow back into planning
- **Verified-done**: goal-backward phase checks, drift detection between plan and git reality, and a ship gate that blocks on open issues
- **Memory with provenance**: lesson cards point at the commits that prove them, get confidence-graded, and get re-graded at retro when reality confirms or contradicts them
- **Hooks**: session continuity across /clear, a leak guard that blocks commits leaking internal refs, per-phase cost attribution

New this week: it's not Claude-only anymore. One installer wires the same server into Grok Build, Copilot CLI, Codex, Gemini CLI, Cursor (full hooks port), OpenCode, and Zed — though Claude Code remains the richest surface (plugin, slash commands, all hooks).

The docs include an honest comparison matrix against GSD, beads, and context-mode — every claim cited to those projects' own issue trackers, and our own gaps documented the same way.

MIT, 744 tests, three runtime dependencies.

Repo: https://github.com/eventually-consistent-code/cairn2
Write-up: https://eventually-consistent.io/projects/cairn/

Feedback very welcome — especially from anyone who's tried to make an agent's self-reported progress trustworthy. That problem is the whole reason this exists…
