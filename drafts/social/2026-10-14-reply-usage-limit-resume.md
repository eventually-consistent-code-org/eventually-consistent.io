# dev.to comment reply — Dawon Lee, "Claude Code now resumes after a usage limit — unless your Mac fell asleep"
# https://dev.to/dawonlee/claude-code-now-resumes-after-a-usage-limit-unless-your-mac-fell-asleep-49dj
# Posting is John's click.

This one found a bug in my docs, so thank you, that's the useful kind of post…

I build cairn, a work harness for Claude Code, and its headless mode runs whole phases unattended inside a token budget, the kind of thing you kick off before bed. The verb file literally says "a usage limit is NOT a kill: the run pauses and resumes itself when the window rolls over, and batch mode carries no defences against limit-time agent loss, because the platform no longer loses them." Which was true the day I wrote it and is exactly wrong on a laptop that sleeps, because `quota_auto_resume_stale` is agent loss at limit time, the run is sitting there waiting for Enter and the tracker issue it was narrating to has gone silent for six hours with no explanation. The three events you dug out of the hooks reference are the fix: a `StopFailure` on `rate_limit` posts "paused, resets at 3:45pm" on the run's tracker issue and writes it into the continuity handoff, `quota_auto_resume_fired` posts "resumed," and `stale` posts "waiting for a human to press Enter," so the person reading the board at 7am sees why nothing moved instead of guessing. Same for `agent_needs_input`, that whole "waiting for you" class belongs on the tracker, not in a terminal nobody's looking at.

Second thing it broke: cairn measures how long an issue took from the claim comment to the close, because the agent's own estimates were running six to fourteen times over reality, and a five-hour reset wait inside that span is dead time the measurement currently eats. Those same two events bound the pause, so it can be subtracted or at least recorded beside the number…

Keeping the Mac awake I'm leaving to you and `caffeinate`, that's the OS's job and a plugin pretending otherwise is how you get a plugin that does `sudo pmset`. Two questions, since you've read the reference closer than I have: does `quota_auto_resume_stale` fire at reset time or at wake time, because a hook that wants to post "waiting for Enter" cares which, and do any of these fire in a `claude -p` headless session, or only in the interactive TUI?

-John Reed
