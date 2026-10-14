# dev.to comment reply — Sagi Tal, "I wrote evals for my Claude Code plugin. The first run proved my README wrong."
# https://dev.to/sagaga144/i-wrote-evals-for-my-claude-code-plugin-the-first-run-proved-my-readme-wrong-jao
# Posting is John's click.

"A failing eval is a claim about either your code or your grader, and you have to check which" is the best one-line summary of the last two weeks of my life, so thanks for that…

I build cairn, a work harness plugin, and we run the same `claude plugin eval` harness over 17 cases, so a lot of this read like my own notes. Where we landed the same: fires-and-holds pairs for every trigger, which is your harmless-twin idea applied to skill descriptions, and the model-free hook tests, 180-odd of them in vitest feeding each guard the JSON payload Claude Code would send and asserting the refusal, including the `git -C` and `cd`-into-a-subfolder shapes, because the guard judges a git command where git would actually run it. And your grader lesson, exactly: our last big eval run started at recall 70% and had to be re-read miss by miss, three of the misses were corpus defects (a mock answering with a canned record, a grader scoring clarity notes as invented defects, ground truth naming the wrong file), the numbers only meant anything after the graders were fixed and the arms rerun.

Two places I think we're a little further along, offered as ideas not as a scorecard. One run per case being noisy is real, so we run three, in two arms, with and without the plugin, and a case whose runs disagree is not "67%," it's INCONCLUSIVE and the cue to add runs, never to average. And cases carry a tier: regression cases must pass all runs and gate, capability cases are allowed to fail and are improvement targets, because mixing the two in one pass rate produces wrong priorities.

Two places you're ahead and I'm taking notes. The per-model matrix is the big one: every cairn eval has run on whatever my account's default model was, and your 8-of-8-was-one-model section is the exact blind spot, I have no idea what our regression tier does on Haiku. How did you pass `--model` through, a runner flag of your own or something the harness supports now? Second, "before finishing, check that every file CLAUDE.md mentions actually exists on disk" is a cheap deterministic guard I don't have in that general form, ours checks verbs against files and tool references against the registry but not arbitrary paths a doc names.

The honest footnote, since you wrote yours: our eval lane in CI is advisory and nightly, allowed to fail, because the suite costs real plan usage and the harness is early access. The seven guards that gate every push recompute counts, not behavior. So the thing that proved your README wrong on the first run would, in my setup, have shown up the next morning as a yellow badge. Which is a confession, not a design…

-John Reed
