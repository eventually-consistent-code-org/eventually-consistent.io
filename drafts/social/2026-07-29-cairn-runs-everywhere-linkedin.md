# LinkedIn — cairn runs everywhere (hold until wave-3 PR merges + site deploys)

Your AI coding agent is brilliant for an afternoon and amnesiac by morning. Cairn fixes that — and as of today, it doesn't care which agent you use.

Cairn started as a Claude Code plugin: plans that live in git, work items mirrored to your real tracker (Jira, GitHub, Linear, +5 more), memory that survives the session, and a verification gate that won't call something done just because closing the issue was easier than testing it.

Today it runs on seven more harnesses: Grok Build, GitHub Copilot CLI, Codex, Gemini CLI, Cursor, OpenCode, and Zed. One installer, three thin layers — MCP wiring, a generated AGENTS.md verb registry, and native slash commands where the harness has them. Cursor even gets the full hook layer: continuity resume, a commit leak guard, observation capture — same scripts, one adapter.

The honest part (we keep an honest capability table, not a checkbox farm): the cost tracker stays Claude-only for now, and every per-harness gap is documented rather than hand-waved.

744 tests. Three runtime dependencies. The tracker paper trail doesn't care which model wrote it…

https://github.com/eventually-consistent-code/cairn2
https://eventually-consistent.io/projects/cairn/

#AIEngineering #DeveloperTools #MCP #AgenticCoding
