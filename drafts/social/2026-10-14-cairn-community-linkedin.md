# LinkedIn — ARTICLE (long-form, not a feed post). Posting is John's click.
# Suggested headline + subtitle below; body follows.

**Headline:** Six Articles In, Four Roadmap Phases Out: What One Night on dev.to Did to My Project

**Subtitle:** The AI tooling community is running one giant experiment in public. Reading it like a bug report against your own code is the best design review money can't buy.

---

I build an open-source work harness for AI coding agents called cairn. For most of the last two months the only thing reviewing my design decisions was my own test suite, which is a fine reviewer for "does it work" and a terrible one for "is this the right shape at all." On October 13th I opened dev.to to unwind, read six articles by six people I've never met, and by the end of the night cairn had four new phases on its roadmap, four issues shipped the same week, and my global rules for every project I touch had three new files in them…

I want to walk through how that happened, because I think the mechanism matters more than the specifics, and I think most teams building with AI right now are leaving it on the table.

## The loop

Here's the whole method. Read the article all the way through. Before forming an opinion, go check your own codebase for the exact thing it describes, down to the line. If the author is right about you, file the fix as real planned work with the article named as its source. If the author is wrong about you, or your design already matches, write that down too, because a confirmed decision with a citation is worth nearly as much as a new one. Then attribute every file that borrows the idea, paraphrase the reasoning, and copy nothing verbatim.

The grounding step is what separates this from inspiration. An idea you admire does nothing for your project until you've found the line where you were wrong.

## What came out

**Alex Aslam wrote about why agents need a constitution, not a prompt.** His point: rules written one incident at a time never converge, you just stay one failure behind. I looked at cairn's run guard and found seventeen issue numbers in its header, seventeen rules each justified by the bug that caused it. He was describing my code. That became phase 28: a short set of axioms that classify what counts as what (the user's data versus scratch, a destructive action, a directive versus data, done versus claimed), with every guard deriving from a named clause and the guard itself rewritten to classify instead of enumerate. The honest line in the phase: this makes refusals derivable and auditable, not airtight. A plugin can't make unsafe paths unreachable by construction, and saying so is part of the design.

**Gabriel Anhaia wrote about why git worktrees break for coding agents.** No `.env`, no dependencies, same port as every other worktree. Cairn had been using worktrees for a while and getting away with it because my own repo happens to need none of that. Luck isn't design. Two issues in phase 30 now derive worktree setup from what the repo declares, never from what the agent guesses, and git hooks join the set of files the agent is refused from editing.

**ThinkerQAQ wrote about blocking queues and channels.** Not an AI article at all. It frames every handoff between two execution units around three questions: atomicity, visibility, ordering. I asked those three of every plain-file handoff in cairn and found a read-trim-rewrite on a live log, a lost-update bug waiting for a second session. Fixed as the first issue of phase 30, and two later phases are sequenced behind it because they build on the same discipline.

**Arun at Offrun wrote a field report on running Claude Code and Codex in parallel on one repo.** His gotcha that landed: a reviewer that only sees tracked files silently misses every new file in the diff. Cairn's review verb had exactly that default. Phase 29 fixes it along with four more mechanics, organized by one rule: whether state belongs to a worktree or the whole repository is decided by what the state is about, not where it's written.

**zeeshani42 built a Claude skill that holds AI-generated UI to a senior designer's bar.** Cairn has a design stage and had no acceptance bar for it. The obvious move was to vendor the skill. I didn't, because cairn's resident context budget had 38 tokens of headroom and the skill's description alone costs about 80. Phase 27 integrates by reference instead: the skill stays upstream under its own license, cairn points at it when installed and behaves identically when it isn't, and a UX review seat joins the panel only when a diff actually touches UI.

**h1manshu01 wrote about picking the least autonomous architecture that solves the problem.** Start with a call, add a workflow when the process needs structure, reserve an agent for when the process has to be discovered. I went looking for places where cairn asked the model to execute a sequence the server could just run. Found four. All four are merged.

## Why this matters beyond my project

Nobody has the full picture of how to build with AI agents yet, and the people who claim to are selling something. What exists instead is a few thousand engineers running the same experiment with slightly different variables and writing up what broke. If you read those write-ups as design reviews of your own assumptions, you get senior-level architectural feedback for free, continuously, from people who have no stake in telling you what you want to hear.

Six articles, six checks against my tree. Three said "the article is right about you." Two named a gap I hadn't put words to. One confirmed my existing direction and still produced four shipped fixes. Even the confirmation earned its place, because the decision now lives in a planning doc with a source instead of in my head.

The other half is that you have to publish too. Somebody is three hours into their own reading night right now, and your post about the thing that broke is the one that fixes their code. Ideas in, ideas out, attribution on everything…

## The sources

- Alex Aslam, "Why your agents need a constitution, not a prompt" — https://dev.to/alex_aslam/why-your-agents-need-a-constitution-not-a-prompt-2181
- Gabriel Anhaia, "Git Worktrees for Coding Agents Break on .env, node_modules and Port 3000" — https://dev.to/gabrielanhaia/git-worktrees-for-coding-agents-break-on-env-nodemodules-and-port-3000-fix-all-three-3781
- ThinkerQAQ, "Concurrency Programming (9): BlockingQueue and Channels" — https://dev.to/thinkerqaq
- Arun (Offrun), "Running Claude Code and Codex in parallel on one repo" — https://dev.to/offrun/running-claude-code-and-codex-in-parallel-on-one-repo-git-worktree-gotchas-and-cross-review-41eg
- zeeshani42, "I built a Claude skill that makes AI design like a senior UX designer" — https://github.com/zeeshani42/your-ui-ux-designer
- h1manshu01, "When to Use an AI Agent Instead of a Simple LLM Call" — https://dev.to/h1manshu01/when-to-use-an-ai-agent-instead-of-a-simple-llm-call-djk

Cairn is MIT-licensed and the roadmap is public: https://github.com/eventually-consistent-code/cairn2

#AIEngineering #AgenticCoding #OpenSource #DeveloperCommunity #ClaudeCode #MCP #DevTools #SoftwareArchitecture #AI #LLM
