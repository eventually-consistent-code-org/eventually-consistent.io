+++
title = "Tailor"
date = 2026-08-03T00:00:00Z
lastmod = 2026-08-17T00:00:00Z
draft = false
flashUpdated = true
tags = ["python", "flask", "aws", "ai", "careers"]
summary = "Paste a job description, get a resume and cover letter tailored to it — grounded in your real experience, written in your voice, nothing invented."
repo = "https://github.com/eventually-consistent-code/tailor"
images = ["img/tailor-icon.jpg"]   # square mark — social-share / og:image
+++

<p style="text-align:center; margin:0 0 2rem;">
  <img src="/img/tailor-logo.jpg" width="220"
       style="max-width:60%; height:auto; border-radius:18px;"
       alt="Tailor logo — a purple gel tile with a serif T formed by a needle and thread, above the wordmark TAILOR">
</p>

You can spot an AI-written resume in about four words — usually "results-driven
professional." Same buzzwords every time, same faint smell of a chatbot that
never met you. Tailor is the antidote. You paste in a job description, and it hands back
a resume and cover letter tuned to that job — grounded in *your* actual
experience, and written in *your* voice, not a model's default one.

Here's the trick that makes it different: grounding is non-negotiable. Tailor
works from a "master resume" — your real career history, every role and bullet
with a stable id. When it tailors a resume to a posting, every line it writes
has to trace back to something you actually did. Anything ungrounded gets
rejected before it ever hits the page. So it reorders, reframes, and emphasizes
to fit the job… but it never invents a skill you don't have or a job you never
worked. No hallucinated accomplishments, ever.

The voice half is the other piece. Instead of one flat "professional" tone,
Tailor learns *yours* — from a short guided interview, from writing samples you
paste in, and over time from the edits you make before you export. It distills
all that into a voice profile that shapes the wording (never the facts). Cover
letters lean on it hardest, which is exactly where the generic-chatbot smell is
worst.

It started life as a single-user tool I ran on my own laptop to stop rewriting
my resume by hand for every application. Over the last stretch it grew up into a
real multi-user web app — a Flask backend with Postgres, Clerk for sign-in,
Stripe for a fair trial-then-pay model, Claude doing the tailoring under prompt
caching to keep per-run cost sane, python-docx exporting a clean `.docx`, all of
it running on AWS behind Terraform. It's live at
[tailor.eventually-consistent.io](https://tailor.eventually-consistent.io).

And it keeps growing. The latest round of updates, roughly in the order
you'll notice them:

- **A whole new look.** Dark obsidian glass, a gel-purple design language,
  and an animated needle-and-thread background — five fine threads drifting
  behind every page. (It respects `prefers-reduced-motion`, because subtle
  should mean subtle.)
- **Resume + cover letter in one pass.** A "Both at once" switch runs the
  pair back-to-back from the same job description, each with its own
  preview, steering, and export.
- **Multiple master resumes.** Keep one per career track — the infra one,
  the security one — and pick which to tailor from per run.
- **Google Docs, both directions.** Import your resume straight from Drive,
  export the finished document back as a native Doc.
- **PDF export** alongside `.docx`, from the same preview.

Who's it for? Anyone tired of the resume treadmill — job switchers,
career-changers, anyone applying to more than one place and unwilling to send
the same generic PDF to all of them. You get a document that's honest to your
history and sounds like you wrote it, in the time it takes to read the posting.

The code is public: [github.com/eventually-consistent-code/tailor](https://github.com/eventually-consistent-code/tailor).
