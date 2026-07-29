# LinkedIn — cairn UPDATE post (v2, changelog style — posting is John's click)

**CAIRN UPDATE** — what shipped in the last 40 hours:

🔹 **Runs everywhere now.** One installer wires cairn into Grok Build, GitHub Copilot CLI, Codex, Gemini CLI, Cursor, OpenCode, and Zed — not just Claude Code. Cursor even gets the full hook layer ported: session continuity, commit leak guard, observation capture. Same scripts, one adapter.

🔹 **On npm.** `npx @eventually-consistent/cairn cairn-setup <harness>` and you're wired. Server publishes with OIDC trusted publishing — no tokens in CI.

🔹 **Cost attribution.** Session spend (tokens + $) now lands on the actual phases and issues it was spent on. Your burndown knows what the agent cost.

🔹 **Custom status vocabularies.** Your board says "In Review" and "Blocked"? Cairn speaks your board's states now across Jira, Linear, ClickUp, Azure, GitHub, and the local tracker — not a hardcoded open/closed.

🔹 **Sprints + estimates.** Issues land in the active sprint with real estimates — actual burndown data, not a graveyard of unestimated tickets.

🔹 **Image attachments.** Diagrams and screenshots flow through to tracker issues and Confluence pages as native attachments.

🔹 **Passive observation capture.** The agent's surprises get recorded as candidate lessons and reviewed at retro — memory that grows from evidence, not vibes.

Still the same spine underneath: plans in git, every piece of work mirrored to your real tracker, and a verification gate that refuses to call things done without evidence. 744 tests. Honest capability table — the gaps are documented, not hand-waved…

🔗 https://github.com/eventually-consistent-code/cairn2
📄 https://eventually-consistent.io/projects/cairn/

#AIEngineering #DeveloperTools #MCP #AgenticCoding
