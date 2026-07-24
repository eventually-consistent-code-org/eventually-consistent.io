+++
title = "Cairn Quick-Start"
date = 2026-07-24T00:00:00Z
draft = false
tags = ["claude-code", "agentic-tooling", "tutorial", "developer-tooling"]
summary = "Zero to a tracker-mirrored project in about fifteen minutes — install cairn, wire up your issue tracker (including every step that happens outside Claude), and run your first phase."
repo = "https://github.com/eventually-consistent-code/cairn2"
+++

What does it actually take to get an AI agent planning your project in the
tracker your team already uses? Less than you'd think — but there are a few
steps that happen *outside* Claude Code, and most guides skip them. This one
doesn't. Fifteen minutes, start to finish, and the finish line is a real
project: plan in git, issues in your tracker, agent ready to work.

If you haven't read [the cairn overview](/projects/cairn/) yet, start there
for the why. This page is the how…

## Step 1 — install the plugin

From any Claude Code session:

```
/plugin marketplace add eventually-consistent-code/cairn2
/plugin install cairn@cairn-dev
```

Then build the MCP server the plugin launches. This is the one honest
speed bump: cairn's server is TypeScript with a native SQLite dependency,
so it needs Node 20+ and one build:

```bash
cd ~/.claude/plugins/cache/cairn*/server   # wherever the plugin landed
npm ci && npm run build
```

Why: the plugin's `.mcp.json` starts `server/dist/index.js`, and `dist/`
only exists after the build. `npm ci` also compiles better-sqlite3's native
bindings for your machine. Do it once, forget it.

## Step 2 — tell cairn where your tracker lives

Drop a `cairn.json` in your project root. If you use GitHub Issues, the
whole thing is four lines:

```json
{
  "tracker": {
    "type": "github",
    "config": { "repo": "you/your-project" }
  }
}
```

Auth happens outside the file: cairn reads `GITHUB_TOKEN` from your
environment, and if that's not set it quietly falls back to `gh auth token`
— so if you've ever run `gh auth login`, you're already done.

One deliberate design choice worth knowing: **cairn.json refuses to hold
secrets.** If a key or value looks like a credential, the config loader
throws. Tokens live in environment variables; the config only names *which*
variable to read. Your tracker config is safe to commit.

### The other five backends

| Backend | Minimal config | Env var(s) | Where the token comes from |
|---|---|---|---|
| GitLab | `project: "group/name"` | `GITLAB_TOKEN` | User Settings → Access Tokens, `api` scope |
| Jira | `baseUrl`, `projectKey`, `transitions: {in_progress, closed}` | `JIRA_EMAIL` + `JIRA_API_TOKEN` | id.atlassian.com → Security → API tokens |
| Asana | `projectGid` | `ASANA_TOKEN` | My Settings → Apps → Personal access tokens |
| Azure Boards | `orgUrl`, `project` | `AZURE_DEVOPS_PAT` | User settings → Personal access tokens, Work Items scope |
| ClickUp | `defaultListId` + `folderId` *or* `spaceId` | `CLICKUP_TOKEN` | Settings → Apps → API token |

The plugin ships `templates/cairn.json.example` with copy-paste blocks for
all six. For Jira, the `transitions` map deserves a sentence: it names your
workflow's transition labels (say, "In Progress" and "Done") so cairn can
move issues through *your* board instead of guessing. Export the env vars
in your shell profile — `export JIRA_EMAIL=you@company.com` and friends —
before starting Claude Code.

## Step 3 — your first project

Open Claude Code in your project and run:

```
/cairn:new my-project
```

Cairn interviews you — what you're building, what done looks like, three to
ten requirements, how the phases should break down. It also asks one
question about *you*: *vibe mode* (cairn drives end-to-end, the default) or
*engineer mode* (you claim issues and write code; cairn pairs, gates, and
never merges its own PRs).

When the interview ends, two things exist:

- **In git:** `.cairn/plans/PROJECT.md` (vision + requirements), a
  `roadmap.md` phase table, and a directory per phase holding `CONTEXT.md`
  for locked decisions and `PLAN.md` for the task breakdown.
- **In your tracker:** a phase (epic, milestone, or list — whatever your
  backend calls it) per roadmap row, and a real issue per requirement,
  each wired into its phase's `PLAN.md` frontmatter.

Your manager can open the board right now and see the whole project. That's
the point.

From there, the loop is three commands:

```
/cairn:plan 1    # research the phase, write the task breakdown
/cairn:work 1    # claim issues, do the work, close with evidence
/cairn:verify 1  # goal-backward check before anything ships
```

Watch the issues as `work` runs: claim comments with base commits, progress
notes at real milestones, close comments with commit ranges and test
evidence. The paper trail writes itself — because if it isn't in the
tracker, it didn't happen…

## The rest of the loadout

Cairn plays well with others — here's what actually runs alongside it in my
own sessions, and one line on why each earns the slot:

- **[superpowers](https://github.com/obra/superpowers)** — process
  discipline as skills: brainstorming before building, systematic debugging
  before guessing, TDD before "trust me."
- **[context-mode](https://github.com/mksglu/context-mode)** — context-window
  economy: heavy command output and file analysis stay in a sandbox instead
  of eating your session. The one job cairn deliberately doesn't do.
- **caveman** — token-diet output modes; the agent talks like a smart
  caveman and your budget thanks you.
- **remember** — lightweight session history that survives `/clear`;
  cheap insurance alongside cairn's own continuity.
- **[context7](https://context7.com)** — live library documentation on
  demand, so the agent stops confidently citing an API from two majors ago.

None of them are required. All of them compound — cairn owns the plan, the
tracker, and the memory; these keep the process sharp and the context
lean…
