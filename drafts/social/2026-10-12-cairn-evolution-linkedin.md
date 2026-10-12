# LinkedIn — cairn UPDATE post (changelog style, Aug → Oct 2026 evolution; posting is John's click)

**CAIRN UPDATE** — two months, eight releases, v2.2 → v2.8.3. Here's what the work harness learned to do:

🔹 **It runs itself, on a budget.** `/cairn:auto --batch` stages whole phases headless: honest cost *ranges* calibrated from real past spend, push authorization granted once up front and scope-limited to exactly those phases, a spend ledger checked at every boundary. First live run shipped seven fixes in 22 minutes against a 13-hour human estimate — and caught its own blind budget meter mid-run. Fixed next release, with a live refusal test.

🔹 **Findings have to survive before they're filed.** Review seats are data now (schema, rubric, anchored scale, required dose), picked by signals in the diff. Every serious finding carries a typed failure scenario and faces a refutation panel voting CONFIRMED / PLAUSIBLE / REFUTED, tallied in code. Refuted majority kills it. `/cairn:audit sweep` runs every audit leg and diffs survivors against the last sweep: new, persisting, fixed, regressed.

🔹 **Guards refuse instead of quietly succeeding.** Three PreToolUse hooks: a leak guard on commits, a run guard that refuses `git reset --hard`, force-pushes, and mid-run pushes without manifest authority (no override), and a harness guard that write-protects the config deciding what the agent may do — and asks where the instruction came from. Written into the design record: these are not a sandbox. They turn a silent success into a refusal you can see.

🔹 **Durations are measured, not asked for.** The agent's self-reported time was running 6–14× over reality (one claimed 128 min against 10 measured). Now the server measures from the claim comment to the close, names which rung produced the number, and keeps the agent's claim beside it — never instead of it.

🔹 **Docs that stop lying.** Deterministic staleness detection over every verified phase (first run found four phases that never made the changelog), generation that can only write inside marked sections, and publishing to Confluence or Docusaurus — cairn's own 78 pages included. Seven CI guards recompute every public count, down to the resident context footprint.

🔹 **The hardening.** v2.8.2 closed 48 issues in 13 waves, re-auditing the security surface after every wave until ten passes came back clean. +385 tests in one release.

The numbers: 75 → 87 typed MCP tools, 1,016 → 2,124 tests, 39 verbs, 8 tracker backends, 8 harnesses, 21 ADRs — and every one of those recomputes itself in CI. Same spine underneath: plans in git, every piece of work mirrored to your real tracker, nothing "done" without evidence…

🔗 https://github.com/eventually-consistent-code/cairn2
📄 https://www.eventually-consistent.io/projects/cairn/

#MCP #ClaudeCode #AgenticCoding #AIEngineering #DeveloperTools #DevTools #AI #ProjectManagement #LLM #OpenSource #AIAgents #Automation
