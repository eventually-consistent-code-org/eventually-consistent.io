# dev.to comment reply (second round) — Dawon Lee, "Claude Code now resumes after a usage limit — unless your Mac fell asleep"
# https://dev.to/dawonlee/claude-code-now-resumes-after-a-usage-limit-unless-your-mac-fell-asleep-49dj
# Replying to his answer (comment-3hop6) on the re-arm limit, the 30-minute stale threshold and the subtract interval. Posting is John's click.
# Verified 2026-10-16 against the interactive-mode and hooks docs; delta posted on cairn2 #379.

The re-arm limit is the one I'd have missed, thank you…

Two hits and then "Automatic continue stopped" means `disabled` is a different animal from the other two, the session is alive and nothing will ever move it again without a human, so on the board it's "stopped, restart by hand" and not "paused," and the morning handoff has to say stopped too or the next session reads it as a pause and waits with it. That's now in the issue, and so is the interval you named, limit stop to `fired`, as the window the duration math subtracts.

Going back to the docs for your two points turned up a third that changes my own claim more than yours did. The wait is interactive only, the page says it outright, background sessions and `-p` runs don't get it at all. Cairn's walk-away batch mode runs inside the TUI session, so it's covered, but anything I launch headless under `claude -p` gets no wait and a rate-limit stop there is just a stop, which answers my second question to you and not in the direction I wanted. Also nothing in either payload carries the reset time, `StopFailure` is error plus details plus the last message, `Notification` is message plus type, so "paused, resets at 3:45pm" has nowhere to read 3:45 from except the message text, and the comment has to say "paused on usage limit" and the time it noticed, with the reset time only when it can parse one. And on stale firing at reset versus at wake, the docs don't say, so the recorder stamps its own clock and never calls that the reset…

That last bit is the thing I keep relearning this week: the docs' sentence and the hook's payload are two different promises, and the one you can build on is the payload.

-John Reed
