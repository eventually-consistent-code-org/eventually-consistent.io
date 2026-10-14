# dev.to comment reply — Anshul, "I Will Rate Your AI Tool Stack. Drop It in the Comments."
# https://dev.to/unfiltered_anshul/i-will-rate-your-ai-tool-stack-drop-it-in-the-comments-13ch
# Goal: get him to actually install and rate cairn. Posting is John's click.

Alright, rate this one, and I want the garbage column filled in, that's the part nobody gives me…

Claude Code is the main driver, same as you. Codex, Grok, Antigravity, and OpenCode are installed too, but I don't drive with them, they sit as reviewers: a council that gets the same evidence packet and has to cite a line in the source before a claim counts. GitHub Issues is the tracker, which sounds boring until you realize it's the only thing in the stack my team actually reads. context-mode keeps tool output out of my context window, caveman keeps my own output short, and the rest of the plugins are small quality-of-life stuff I'd drop tomorrow without noticing.

The one I actually want you to beat up is cairn, because I built it and I'm the worst possible judge of it. It's a work harness for Claude Code (and seven other CLIs): the plan lives in git, every work item mirrors into your real tracker, memory cards carry provenance and come back flagged STALE when the code moved, and nothing closes as done without typed evidence, the server measures the time instead of asking the agent, because the agent's own estimates were running six to fourteen times over reality. Headless mode runs whole phases inside a token budget you set, and three hooks refuse the dumb stuff (force-push, hard reset, editing its own guard config) with one line telling you why.

What sucks, since you asked: it costs about 13k tokens of resident context just to have installed, the hooks only fully work on Claude Code (Cursor partial, nothing elsewhere), setting up a tracker is a real step with credentials and it's the step people quit at, and 39 verbs is more than anyone should have to learn, even though `/cairn:do "whatever you meant"` routes for you. Every one of those is written in the README instead of hidden, which I'd like to think counts for something.

Two commands in Claude Code and you're in:

```
/plugin marketplace add eventually-consistent-code/cairn2
/plugin install cairn@cairn-dev
```

Then `/cairn:new` on a throwaway repo with the zero-credential local tracker, so you skip the credentials step entirely, and run one phase through plan → work → verify. Rate the stack on whatever scale you want, but rate cairn on one question: did the tracker and the plan ever disagree, and did it catch that or did you? If you find it's a 4, I want the 4 and the reason, that's a tracker issue I'll file the same day…

https://github.com/eventually-consistent-code/cairn2

-John Reed
