# r/ClaudeAI draft v2 (posting is John's click)

Automod-safe shape: NO links in the post body (repo link goes in your own
first comment after the post is live), no feature-bullet wall, no "check it
out" phrasing — reads as an experience/discussion post, which is what it is.

**Title:** What finally fixed "brilliant for an afternoon, amnesiac by morning" for me: making the agent live in my actual issue tracker

**Body:**

I kept hitting the same wall with long-running Claude Code projects. The
agent plans beautifully, works hard for a session... and then the context
window rolls over, or I /clear, and the plan is gone. Chat scrollback
became my project management system. Meanwhile my actual Jira board — the
one other humans look at — knew nothing.

The fix that stuck for me wasn't better prompting, it was moving the
source of truth out of the conversation entirely. Everything the agent
plans or ships gets mirrored to the real tracker as it happens: issues,
comments, worklogs, estimates. The plan lives in git. When a session dies
mid-task, the next one picks up from the tracker + a handoff file, not
from vibes.

The part I underestimated: verification. Agents will absolutely close
their own tickets because closing is easier than testing. What worked was
a goal-backward gate — the phase isn't done because issues got closed;
issues get closed because a verifier walked backward from the goal and
found evidence. Plus drift detection between what the plan claims and
what git actually shows.

The other surprise was memory. Storing "lessons learned" as freeform
notes rotted fast. What held up: lessons as cards pointing at the exact
commits that prove them, with a confidence grade that gets re-checked
later — confirmed lessons grade up, contradicted ones grade down and get
corrected. Stale memory confidently outranking fresh reality is a real
failure mode; I've watched other tools in this space wrestle with it.

I ended up building all this into an open-source plugin (MIT). It also
turned out the same server runs under Grok, Copilot CLI, Codex, Gemini,
Cursor, OpenCode, and Zed since the whole thing is just MCP + markdown.
I'll drop the repo in a comment for anyone who wants to poke at it.

Curious what others do for this — especially: has anyone found a way to
trust an agent's self-reported "done" without an independent check? That
problem shaped basically every design decision here.

**First comment (post after the thread is live):**

Repo (MIT): https://github.com/eventually-consistent-code/cairn2 — the
docs include an honest comparison against GSD/beads/context-mode with
citations to their own issue trackers, and our own gaps documented the
same way (e.g. cost tracking is Claude Code-only, and the docs say why).
