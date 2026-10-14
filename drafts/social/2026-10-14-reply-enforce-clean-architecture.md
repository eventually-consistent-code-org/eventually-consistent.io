# dev.to comment reply — Arg Software, "How to Actually Enforce Clean Architecture in TypeScript"
# https://dev.to/argsoftware/how-to-actually-enforce-clean-architecture-in-typescript-1o4a
# Posting is John's click.

Your "silent killer" section is the part I'd put at the top, because it's the one nobody arrives at until it bites them…

I build cairn, a work harness for coding agents, and the TypeScript server under it has had a hand-rolled version of this since September: tracker adapters are leaves, the tracker subsystem imports none of the subsystems that read it, the docs connectors may reuse four named tracker modules and nothing else, `index.ts` is the composition root nobody imports back, and the whole tree has no cycles. About 170 lines of vitest walking the import specifiers, no library. The header comment on that file says almost word for word what your section says, a rule that inspected zero files and passed is worse than no rule, so every rule goes through one `filesMatching` helper that refuses to run on an empty set. Nice to see ArchUnitTS make that the default instead of a thing you have to remember to write.

One thing we do that I'd suggest adding to the article, since it's the same disease one step later: the allowlist entries have to earn their keep too. The docs-may-reuse-these-four-modules list has a second test that fails when any entry is no longer imported by anything, so an exception that stopped being needed gets deleted instead of sitting there as a hole for the next person. The empty pattern is a rule that guards nothing, the dead exception is a rule that permits something nobody asked for, and both read as coverage forever.

And the honest part, because reading your piece made me go check: the plugin layer around that server has one layering rule of its own, the hook scripts import no server code so the server never becomes a dependency of the thing that guards it, and that rule lived in exactly the place you warn about, a paragraph in ARCHITECTURE.md. True today, asserted nowhere. Filed the test this afternoon. So, one article, one gap found, which is about the rate I'd hope for…

The one place I'd push back a little is "stop relying on PR reviews." The import rules are the cheap tenth of architecture, the expensive nine tenths is whether the thing in the application folder is actually a use case or a controller with a different suffix, and no file-pattern test sees that. The test earns its place by making the review not have to look at imports at all, so the human spends their attention where the pattern can't…

-John Reed
