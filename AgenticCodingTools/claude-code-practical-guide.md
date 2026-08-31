# Claude Code — A Practical Reference

> Personal study notes and working reference for **Claude Code**, Anthropic's agentic
> coding tool. Rebuilt and expanded from the Udemy course
> *["Claude Code: The Practical Guide Boost your agentic engineering game by mastering
> Claude Code's basic and advanced features"](https://www.udemy.com/course/claude-code-the-practical-guide)*,
> then cross-checked and extended against the official documentation
> ([code.claude.com/docs](https://code.claude.com/docs)).
>
> Claude Code moves fast. Treat this as a durable mental model rather than a spec,
> when a detail matters, confirm it in the docs. Last reviewed: **August 2026**.

---

## Table of contents

1. [What Claude Code is, and where it runs](#1-what-claude-code-is-and-where-it-runs)
2. [Models and effort](#2-models-and-effort)
3. [Prompt & context engineering](#3-prompt--context-engineering)
4. [Starting and steering a session](#4-starting-and-steering-a-session)
5. [Memory: making Claude remember your project](#5-memory-making-claude-remember-your-project)
6. [Permissions, auto mode & sandboxing](#6-permissions-auto-mode--sandboxing)
7. [Extending Claude Code](#7-extending-claude-code)
8. [Feedback loops: browser & testing](#8-feedback-loops-browser--testing)
9. [Parallelism and automation](#9-parallelism-and-automation)
10. [Claude Code on the web (cloud)](#10-claude-code-on-the-web-cloud)
11. [Dependencies and housekeeping](#11-dependencies-and-housekeeping)
12. [Command cheat sheet](#12-command-cheat-sheet)
13. [Sources](#13-sources)

---

## 1. What Claude Code is, and where it runs

Claude Code is an *agentic* coding tool. Unlike a chat window that answers and waits, it
reads your files, runs commands, edits code, and works through a problem while you watch,
redirect, or step away. You describe the outcome: Claude explores, plans, and implements.

### Surfaces

The same underlying engine runs across several surfaces, and they share your repo's
`CLAUDE.md` files, settings, and MCP servers, so configuration you write once follows you
everywhere.

| Surface | Best for |
| --- | --- |
| **Terminal CLI** | The full-featured default. Scriptable, pipeable, Unix-friendly. |
| **VS Code / Cursor extension** | Inline diffs, `@`-mentions, plan review, and history in the editor. |
| **JetBrains plugin** | IntelliJ, PyCharm, WebStorm, etc. Runs the CLI in the IDE terminal. |
| **Desktop app** | Visual diff review, several sessions side by side (each in its own git worktree), scheduled tasks, and cloud sessions. Bundles the CLI. |
| **Web** (`claude.ai/code`) | No local setup. Long-running/parallel tasks in the cloud (see §10). |
| **Mobile** (iOS/Android) | Kick off, monitor, and steer tasks from your phone. |

### Installation (terminal)

The **native installer** is now recommended and auto-updates in the background. It ships as
a native binary, so you do **not** need a separate Node.js install for this path.

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex
```

Other package managers work too (these do **not** auto-update — upgrade them yourself):

```bash
brew install --cask claude-code          # Homebrew (macOS)
winget install Anthropic.ClaudeCode      # Windows
npm install -g @anthropic-ai/claude-code # npm — this path needs Node.js 18+
```

Then start it in any project:

```bash
cd your-project
claude
```

You'll be asked to sign in on first use (a Claude subscription or an Anthropic Console
account). On native Windows, installing *Git for Windows* lets Claude use the Bash tool;
without it, Claude falls back to PowerShell.

---

## 2. Models and effort

You rarely need to think about this, but it's worth knowing the levers.

- **`/model`** switches the active model mid-session. Current families include **Opus**
  (deep, complex agentic work), **Sonnet** (frontier intelligence at scale, the default for
  most sessions), and **Haiku** (fastest, cheapest). Anthropic's highest tier, **Fable**, is
  aimed at long, autonomous runs in large repositories.
- **Effort** controls how much reasoning Claude spends per step: `low`, `medium`, `high`,
  `xhigh`, and `max` (the deepest, sometimes surfaced as "ultracode"). Set it with the model
  configuration or per-skill/per-subagent frontmatter. Higher effort costs more tokens and
  latency, so reserve the top levels for genuinely hard problems.
- **Fast mode** trades a little quality for quicker responses on capable models.
- One-off deep reasoning: include the word **`ultrathink`** in a prompt or skill to request
  a deeper think for that turn only.

---

## 3. Prompt & context engineering

This is the highest-leverage skill. A good session starts with a good prompt.

> **The core equation:** *specific instruction(s)* **+** *relevant context*.

### Start from a specification

For anything non-trivial, give Claude a spec document rather than a one-liner. If you don't
have one, you can generate a first draft by describing your app to an LLM and asking for a
technical specification, then refine it. Even on an existing codebase, describe the app,
its architecture, tech stack, and constraints.

A powerful pattern for larger features is to let Claude write the spec *with* you:

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask
obvious questions dig into the hard parts I might not have considered. Keep interviewing
until we've covered everything, then write a complete spec to SPEC.md.
```

Then start a **fresh session** to implement `SPEC.md` with clean context. The most useful
specs name the files and interfaces involved, state what is out of scope, and end with an
end-to-end verification step that proves the feature works.

### Reference files with `@`

Point Claude at the exact files that matter instead of describing where code lives. Claude
reads them before responding:

```text
We are using the app described in @SPEC.md. Follow the widget pattern in @HotDogWidget.php
to add a new calendar widget.
```

You can also paste/drag images and screenshots directly, pipe data in
(`cat error.log | claude`), or give documentation URLs.

### Recommendations distilled

- **Be concise and precise.** Describe the task clearly and completely, but cut fluff and
irrelevant detail. Noise crowds out the instructions that matter.

- **No unnecessary context.** Useful context is crucial. Irrelevant context is
counterproductive. Reference the files you *know* matter, not the ones you *think* might.
The same goes for docs, if it isn't relevant, leave it out.

- **Think, then plan, then prompt.** Resist typing first and "fixing it over time." If you
find yourself constantly clarifying or asking for follow-ups, invest more upfront. Use
**plan mode** for anything beyond the trivial.

- **Don't "test" the AI.** If you know a task has a tricky part, a common pitfall, or a
gotcha, *say so*, and include the recommended solution. Watching the model stumble on
something you could have flagged wastes everyone's time.

- **Name the tools it should use.** Claude has many capabilities: Bash, web access, MCP
servers, subagents, skills. If a specific tool or feature should handle part of the task,
tell Claude explicitly. Don't hope it picks the right one on its own.

- **Give it a way to verify.** The single biggest upgrade to unattended work: hand Claude a
check it can run. A test suite, a build, a linter, a screenshot to diff against a design.
Without a verifiable signal, "looks done" is the only thing to go on, and *you* become the
verification loop. With one, Claude iterates until the check passes.

| Instead of… | Prefer… |
| --- | --- |
| "add tests for foo.py" | "write a test for foo.py covering the logged-out edge case; avoid mocks; run it after" |
| "make the dashboard look better" | "[screenshot] implement this design, screenshot the result, diff it, fix the differences" |
| "the build is failing" | "the build fails with [error]; fix the root cause, don't suppress it, and verify the build passes" |
| "fix the login bug" | "login fails after session timeout; check token refresh in src/auth/; write a failing test, then fix it" |

---

## 4. Starting and steering a session

### `/init`

Run `/init` once in a project. Claude explores the codebase and generates a starter
`CLAUDE.md` with build commands, test instructions, and the conventions it discovers. If a
`CLAUDE.md` already exists, `/init` proposes improvements rather than overwriting it. Refine
from there with the things Claude *couldn't* infer.

A good early addition to `CLAUDE.md`:

```markdown
We are building the app described in @SPEC.md. Read that file for architecture, the exact
database structure, and the tech stack.

Keep replies concise and focused on key information. No unnecessary fluff, no long code
snippets unless asked.
```

### Plan mode

Use **plan mode** for most non-trivial tasks. Claude reads files and proposes changes
*without* editing your source until you approve a plan. The recommended rhythm is four
phases:

1. **Explore** — `Shift+Tab` until the status bar shows `⏸ plan mode on` (or launch with
   `claude --permission-mode plan`). Ask it to read the relevant code.
2. **Plan** — ask for a detailed implementation plan. `Ctrl+G` opens the plan in your editor
   to tweak it directly.
3. **Implement** — approve the plan (or `Shift+Tab` out) and let Claude code against it.
4. **Commit** — ask it to commit with a good message and open a PR.

Skip planning for genuinely small changes (a typo, a log line, a rename): the overhead
isn't worth it. Plan when the approach is uncertain, the change spans multiple files, or the
code is unfamiliar.

### Course-correct early and often

The best results come from tight feedback loops:

- **`Esc`** — stop Claude mid-action; context is preserved so you can redirect.
- **`Esc Esc`** or **`/rewind`** — open the rewind menu (see below).
- **"Undo that"** — have Claude revert its own changes.
- **`/clear`** — reset context between unrelated tasks.

If you've corrected Claude more than twice on the same issue, the context is polluted with
failed attempts. `/clear` and restart with a sharper prompt that folds in what you learned,
a clean session with a better prompt almost always beats a long, cluttered one.

### Checkpointing & `/rewind`

Claude automatically snapshots files before each change. Every prompt you send creates a
checkpoint. Open the rewind menu with **`Esc Esc`** (empty input) or **`/rewind`** and choose:

- Restore **code and conversation** to that point
- Restore **conversation** only (keep current code)
- Restore **code** only (keep the conversation)
- **Summarize from / up to here** compress part of the conversation to free context

Checkpoints are saved with the session, so you can close the terminal, resume later, and
still rewind. This makes it cheap to try something risky and roll back if it doesn't work.

> ⚠️ Checkpoints only track edits made through Claude's **file-editing tools**. Changes made
> by Bash commands, subagents running in the background, or external processes are **not**
> captured. It is not a replacement for Git.

### Managing sessions

Sessions are persistent. Name them and treat them like branches:

```bash
claude --continue        # resume the most recent session
claude --resume          # pick from a list
/rename oauth-migration  # give the current session a memorable name
```

---

## 5. Memory: making Claude remember your project

Each session starts with a fresh context window. Two mechanisms carry knowledge across
sessions: **`CLAUDE.md`** (instructions you write) and **auto memory** (notes Claude writes
itself). Both are loaded at the start of every conversation, they're *context*, not hard
enforcement. For something that must happen every time, use a hook instead.

### `CLAUDE.md`

Plain-markdown instructions Claude reads at session start. Keep it **short** (target under
~200 lines), specific, and well-structured. A bloated file causes Claude to ignore the
rules that matter.

Files can live at several scopes, loaded from broadest to most specific:

| Scope | Location | Shared with |
| --- | --- | --- |
| Managed policy (org) | OS-specific system path | Everyone on the machine |
| User | `~/.claude/CLAUDE.md` | Just you, all projects |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team, via version control |
| Local | `./CLAUDE.local.md` (gitignore it) | Just you, this project |

You can have **multiple** `CLAUDE.md` files. Those above your working directory load at
launch; those in subdirectories load on demand when Claude touches files there. Import other
files with `@path/to/file` syntax.

The conciseness test for every line: *"Would removing this cause Claude to make a mistake?"*
If not, cut it.

| ✅ Include | ❌ Exclude |
| --- | --- |
| Bash commands Claude can't guess | Anything derivable from reading the code |
| Code style that differs from defaults | Standard conventions Claude already knows |
| Test instructions / preferred runner | Detailed API docs (link instead) |
| Repo etiquette (branch/PR conventions) | Info that changes frequently |
| Architecture decisions specific to you | File-by-file descriptions of the codebase |
| Non-obvious gotchas | "Write clean code" and other truisms |

> **Compatibility note:** Claude Code reads `CLAUDE.md`, not `AGENTS.md`. If your repo already
> uses `AGENTS.md`, create a `CLAUDE.md` that imports it (`@AGENTS.md`) so both tools stay in
> sync. `/init` also reads Cursor and Copilot rule files and folds relevant parts in.

### `.claude/rules/`

For larger projects, split instructions into topic files under `.claude/rules/`
(`testing.md`, `api-design.md`, …). The key advantage: **path-scoped rules** only load when
Claude works with matching files, saving context:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API rules
- All endpoints must validate input
- Use the standard error response format
```

Rules without a `paths` field load every session, at the same priority as
`.claude/CLAUDE.md`.

### Auto memory

On by default. As Claude works, it quietly saves learnings it couldn't derive from the code, 
your role and preferences, corrections you gave it, ongoing project context, and where to
find external references. It's plain markdown stored per-repository; browse or edit it with
**`/memory`**. Toggle it there, or set `autoMemoryEnabled: false` in settings to disable.

You can also just tell Claude to remember things ("always use pnpm, not npm") and it will
save them.

---

## 6. Permissions, auto mode & sandboxing

A **permission mode** decides whether Claude asks before acting. Cycle modes with
`Shift+Tab`.

| Mode | Runs without asking | Best for |
| --- | --- | --- |
| **Manual** (`default`) | Reads only | Sensitive work, reviewing every action |
| `acceptEdits` | Reads + file edits + common fs commands | Iterating on code you review after |
| `plan` | Reads (edits blocked until you approve a plan) | Exploring before changing |
| **`auto`** | Everything, with background safety checks | Long tasks, reducing prompt fatigue |
| `dontAsk` | Only pre-approved tools | Locked-down CI |
| `bypassPermissions` | Everything, no checks | **Isolated containers/VMs only** |

### Auto mode

The standout modern feature, and the default starting mode on Pro/Max/Team plans. Instead of
prompting *you* before each action, a **separate classifier model** reviews actions before
they run. It lets routine work through and blocks things that look dangerous, scope
escalation, unrecognized infrastructure, production deploys/migrations, `curl | bash`, force
pushes, mass deletions, sending secrets outside the repo, and more. Explicit `ask` rules
still prompt you.

You can state boundaries in plain language ("don't push until I review") and the classifier
enforces them. If it blocks something wrongly, it pauses and hands control back to you.

```bash
claude --permission-mode auto -p "fix all lint errors"
```

> Auto mode reduces prompts; it does **not** guarantee safety. Use it where you trust the
> general direction, not as a substitute for review on sensitive operations.

### Sandboxing

Independent of permission modes, the **sandboxed Bash tool** adds OS-level filesystem and
network isolation so commands run within defined boundaries. Enable it with `/sandbox` or
`"sandbox": { "enabled": true }` in settings. For fully autonomous runs, combine a sandbox
(or a container/VM) with auto mode. Pre-approve trusted commands with `/permissions` to cut
prompts further.

### Pre-approving tools

Instead of clicking "yes" repeatedly, allowlist commands you trust with `/permissions`
(e.g. `Bash(npm test)`, `git commit`). Deny rules block in *every* mode, including
`bypassPermissions`.

---

## 7. Extending Claude Code

Five mechanisms, each with a sweet spot. Choose by *when the guidance should apply* and
*whether it must be enforced*:

| Mechanism | Use it for | Loaded |
| --- | --- | --- |
| **`CLAUDE.md` / rules** | Always-on facts and conventions | Every session (rules can be path-scoped) |
| **Skills** | Reusable knowledge or procedures | On demand, when relevant or invoked |
| **Subagents** | Isolated, focused tasks in their own context | When delegated |
| **Hooks** | Actions that must happen *every time* | Deterministically, on lifecycle events |
| **MCP servers** | Connecting external tools/data | When their tools are called |
| **Plugins** | Bundling all of the above to share | When installed/enabled |

### Skills

> **Important update:** custom **commands and skills have merged.** A file at
> `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create
> `/deploy` and behave the same. Existing `.claude/commands/` files keep working, but skills
> are now the recommended form, they support a directory of supporting files, richer
> frontmatter, and automatic (model-decided) invocation.

A skill is a `SKILL.md` file with YAML frontmatter plus markdown instructions. Claude loads
it automatically when the description matches, or you invoke it directly with `/skill-name`.
Because a skill's body only loads when used, long reference material costs almost nothing
until you need it — a big advantage over stuffing everything into `CLAUDE.md`.

Where skills live (higher scope wins on name conflicts):

| Location | Applies to |
| --- | --- |
| `~/.claude/skills/<name>/SKILL.md` | All your projects |
| `.claude/skills/<name>/SKILL.md` | This project only (commit it to share) |
| Plugin `skills/` directory | Where the plugin is enabled |

Minimal example, only `description` is really needed (name defaults to the directory):

```markdown
---
description: API design conventions for our services. Use when writing or reviewing endpoints.
---
- Use kebab-case for URL paths and camelCase for JSON properties
- Always paginate list endpoints
- Version APIs in the path (/v1/, /v2/)
```

A procedural skill you trigger manually, with pre-approved tools and arguments:

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true          # only you can run it
allowed-tools: Bash(gh *) Bash(git *)   # no per-use approval this turn
---
Analyze and fix GitHub issue $ARGUMENTS following our standards.
1. `gh issue view` to read it   2. find relevant files   3. implement the fix
4. write and run tests          5. commit, push, open a PR
```

Run it with `/fix-issue 1234`.

Useful frontmatter fields: `description` (drives auto-invocation put the key use case
first), `disable-model-invocation` (manual-only), `allowed-tools` / `disallowed-tools`,
`argument-hint`, `model`, `effort`, `paths` (auto-load only for matching files), and
`context: fork` (run the skill in its own subagent context).

**Dynamic context injection** run a shell command *before* Claude sees the skill and inline
its output, so instructions arrive grounded in live data:

````markdown
## Current changes
!`git diff HEAD`

## Task
Summarize the diff above in a few bullets and flag anything risky.
````

Handy **bundled skills** that ship with Claude Code: `/doctor` (setup checkup),
`/code-review` (multi-agent diff review), `/run` and `/verify` (launch and confirm your app
works), `/batch` (fan a change out across subagents), `/debug`, and `/loop`.

> If Claude isn't triggering your skill, tune the `description` to include words you'd
> actually say; if it triggers too often, make the description more specific or set
> `disable-model-invocation: true`.

### Subagents

A subagent is a specialized assistant that runs in its **own context window** with its own
system prompt, tool access, and permissions. Use one when a side task (reading many files,
running a noisy test suite) would otherwise flood your main conversation, the subagent does
the work and returns only a summary. This is the primary tool for **preserving context**.

Create one by adding a markdown file to `.claude/agents/` (project) or `~/.claude/agents/`
(all projects). The easiest way is to ask Claude to write it.

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use proactively after changes.
tools: Read, Grep, Glob, Bash   # restrict what it can touch
model: sonnet
---
You are a senior security engineer. Review for injection flaws, auth/authorization issues,
secrets in code, and insecure data handling. Give line references and suggested fixes.
```

Then delegate explicitly: *"use the security-reviewer subagent to check the auth changes"* 
or `@`-mention it to guarantee it runs.

Built-in subagents Claude uses automatically: **Explore** (fast, read-only codebase search),
**Plan** (research during plan mode), and **general-purpose** (multi-step work). Explore and
Plan skip `CLAUDE.md` and git status to stay cheap.

Key ideas:

- **Restrict tools** with `tools` (allowlist) or `disallowedTools` (denylist).
- **Persistent memory:** add `memory: project` so a subagent accumulates learnings across
  sessions.
- **Adversarial review:** a reviewer running in a fresh subagent sees only the diff, not the
  reasoning that produced it, so it judges the work on its own terms. Tell it to flag only
  gaps that affect correctness or stated requirements: a reviewer asked to find problems
  will always find some.

> Course tip made concrete: a documentation-lookup subagent (e.g. a `DocsExplorer` that
> checks official docs before using a third-party library) is a great pattern. Encourage it
> in `CLAUDE.md`: *"Whenever working with any third-party library, look up the official docs
> first using the DocsExplorer subagent."*

### Hooks

Hooks run **shell commands automatically** at specific lifecycle events. Unlike `CLAUDE.md`
guidance, hooks are deterministic, they guarantee the action happens. Configure them in
`.claude/settings.json`, or ask Claude to write one ("write a hook that runs eslint after
every file edit"). Browse configured hooks with `/hooks`.

Example: auto-format after every edit or write:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "cd \"$CLAUDE_PROJECT_DIR\" && npm run format 2>/dev/null || true" }
        ]
      }
    ]
  }
}
```

Common events: `PreToolUse` (validate/block before a tool runs, exit code `2` blocks it),
`PostToolUse` (react after), `Stop` / `SubagentStop` (when a turn or subagent finishes),
`SubagentStart`. A `PreToolUse` hook is the right tool for a hard guarantee ("never let a
write touch the migrations folder"), which no `CLAUDE.md` instruction can promise.

### MCP (Model Context Protocol)

MCP is an open standard for connecting Claude Code to external data and tools: Google Drive,
Figma, Slack, Jira, GitHub, databases, or your own custom tooling. Add a server once and its
tools become available to Claude:

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp login <server>   # authenticate
/mcp                        # see connected servers and their status
```

You can also scope an MCP server to a single subagent via its `mcpServers` frontmatter, so
its tool descriptions don't consume context in the main conversation. Prefer plain CLI tools
(`gh`, `aws`, `gcloud`) when they exist. They're the most context-efficient way to reach a
service.

### Plugins

Plugins bundle skills, subagents, hooks, and MCP servers into a single installable unit.
Browse and install from the marketplace with **`/plugin`**:

```text
/plugin                                              # open the marketplace UI
/plugin install <name>@<marketplace>                 # install one
/plugin marketplace add anthropics/claude-plugins-official
```

Community skills and plugins are also indexed at [skills.sh](https://www.skills.sh/). If you
work in a typed language, a **code-intelligence** plugin gives Claude precise symbol
navigation and error detection after edits.

---

## 8. Feedback loops: browser & testing

### Browser access (front-end work)

Close the loop on UI work by letting Claude *see* the result. Connect it to Chrome (or use a
Playwright MCP/plugin) and ask it to drive the app:

```text
Use Playwright to test all the main features step by step. The dev server is running on
port 3001. Take screenshots, compare against the design, and fix any differences.
```

Claude can also read console logs, fill forms, and extract data from pages. Screenshots are
first-class input: paste one and say *"see the attached screenshot."*

### Testing

Give Claude the task of writing and running unit tests with your project's framework (Vitest,
Jest, pytest, …). This is the verification signal from §3 in practice, the test suite is the
check Claude iterates against. The bundled `/verify` and `/run` skills go further: they build
and launch your actual app to confirm a change works, rather than trusting tests alone.

---

## 9. Parallelism and automation

Once you're effective with one Claude, multiply the output.

### Parallel and background work

- **Subagents**: delegate research/verification to keep your main context clean (see §7).
- **Worktrees**: run several CLI sessions in isolated git checkouts so edits don't collide
  (`claude --worktree`, or the Desktop app manages this visually).
- **Background agents / agent view**: dispatch many sessions that keep running and watch
  them from one screen (`claude agents`).
- **Dynamic workflows**: orchestrate many subagents from a script Claude writes and you can
  rerun; good for large migrations, codebase audits, and cross-checked research.
- **`/batch <instruction>`**: split a change across 5–30 subagents, each in its own worktree
  opening its own PR.
- **Cloud sessions**: offload work to Anthropic-managed infrastructure entirely, and run
  many tasks in parallel across repos without juggling local worktrees (see §10).

A quality pattern regardless of scale: a **Writer/Reviewer** split: one session implements,
a second reviews in fresh context (a clean reviewer isn't biased toward code it just wrote).

### Headless / non-interactive mode

`claude -p "prompt"` runs Claude without the interactive UI: the backbone of CI, pre-commit
hooks, and scripted fan-out:

```bash
claude -p "Explain what this project does"                       # plain text
claude -p "List all API endpoints" --output-format json         # structured
claude -p "Analyze this log" --output-format stream-json --verbose
```

Fan out across files by looping, and scope permissions tightly for unattended runs:

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from Python 2 to 3. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

### Scheduling and long-running loops

Modern, first-class alternatives to hand-rolled loops:

- **`/loop`**: repeat a prompt within a session for quick polling or iteration.
- **`/goal`**: set a completion condition; a separate evaluator re-checks after each turn
  and Claude keeps working until the goal holds (or is judged impossible).
- **Routines**: scheduled tasks that run in the **cloud** (so they run even with your
  machine off), and can also trigger on API calls or GitHub events. Create with `/schedule`.
- **Desktop scheduled tasks**: recurring tasks that run **locally** with access to your
  files and tools.

### The "Ralph loop" — and its modern equivalents

The course's *Ralph loop* is a shell script that repeatedly re-invokes Claude Code (with all
permissions allowed, in a sandbox), each time picking a task from a task list, doing it, and
testing it. Then you run `./ralph.sh <max-iterations>`. It can produce good prototyping
results but costs a lot of tokens, offers less control, and requires reviewing all generated
output. The task list can itself be generated by AI from a specification:

```json
[
  {
    "category": "ui",
    "description": "Build sign-up form component",
    "steps": ["Create components/SignUpForm.tsx client component"],
    "passes": false
  }
]
```

Before running anything like this, enable sandboxing (`"sandbox": { "enabled": true }`) and
run inside a container/VM.

**In 2026, most of what Ralph did is now built in and safer:** `auto` mode provides
classifier-reviewed autonomy without blanket `bypassPermissions`; **`/goal`** keeps Claude
working toward a condition; **routines** and **scheduled tasks** handle repetition; and
**`/batch`** / dynamic workflows fan work out across subagents. Reach for those before
writing a raw loop, you keep the autonomy while regaining control and safety. Whichever you
use, the caveats stand: **review all generated code**, and expect token costs to add up.

### CI and chat integrations

- **GitHub Actions** / **GitLab CI/CD**: respond to `@claude` mentions, turn issues into
  PRs, automate review.
- **Automatic PR code review**: multi-agent analysis of your full codebase on every PR.
- **Slack**: mention `@Claude` with a bug report and get a PR back.

### Working from anywhere

Sessions aren't tied to one surface. Continue a local session from your phone with **Remote
Control**; start a task on the web/mobile and pull it into your terminal with
`claude --teleport`; or run `/desktop` to hand a terminal session to the Desktop app for
visual diff review. The next section covers delegating full tasks to the cloud.

---

## 10. Claude Code on the web (cloud)

Claude Code on the web runs tasks on **Anthropic-managed cloud infrastructure** instead of
your machine. You describe a task in the browser (or on your phone); Claude clones your
GitHub repo into an isolated VM, works on a branch, and pushes it back for you to review.
Nothing is checked out locally, no terminal stays open, and **sessions keep running after you
close the tab**. You can start a task on your laptop and review it later from your phone.

It shines for: several **independent tasks in parallel** (each its own session and branch),
**repos you don't have checked out locally**, well-defined tasks that **don't need constant
steering**, and codebase questions/exploration. For work that needs your local config,
tools, or environment, run locally or use **Remote Control** instead.

> **Availability:** research preview for Pro, Max, and Team plans, and for Enterprise users
> with premium or Chat + Claude Code seats. **GitHub is required**, GitLab and other
> non-GitHub repos can't be used with cloud sessions (you can, however, bundle a *local* repo
> into a cloud session with `claude --cloud`). Zero Data Retention orgs can't use cloud
> features.

### How a cloud session runs

1. **Clone & prepare**: your repo is cloned to a managed VM; your setup script runs if
   configured.
2. **Configure network**: internet access is set by your environment's access level.
3. **Work**: Claude analyzes, edits, runs tests, and checks its work. Watch and steer, or
   step away.
4. **Push the branch**: at a stopping point Claude pushes its branch to GitHub. The session
   *stays open*: review the diff, leave inline comments, create a PR, or message it to keep
   going, all in the same conversation.

### Connect GitHub (one-time)

**From the browser:**

1. Go to **[claude.ai/code](https://claude.ai/code)** and sign in with your Claude account.
   On macOS/Windows the first screen offers the Desktop app; click **Continue on web** to
   stay in the browser.
2. Follow the **Sign in with GitHub** prompt and approve the authorization on GitHub. A cloud
   session can reach any repo your GitHub account can see. (To start a brand-new project,
   create an empty repo on GitHub first.)
3. Optionally **install the Claude GitHub App** on the repos you want *auto-fix* for (see
   below). If you don't need it, click **Skip**, sessions still reach the same repos.
4. **Set up your Default environment.** Pro/Max get a `Default` environment created for you;
   Team/Enterprise fill in a short "create your first environment" form. `Default` uses
   **`Trusted`** network access (reaches common package registries and allowlisted domains,
   nothing else) and works as-is for a first project.

**From the terminal** (if you already use the GitHub CLI, no browser needed):

```bash
gh auth login       # authenticate the GitHub CLI (if you haven't)
claude              # start the CLI
/login              # sign in with your claude.ai account (not an API key)
/web-setup          # syncs your gh token to Claude + creates the Default environment
```

On success it prints `Connected as <your-github-username>` and opens claude.ai/code.

> **Team/Enterprise note:** an **Owner** must first turn on the GitHub connector under
> *Admin settings → Connectors*. A separate optional *Quick web setup* toggle enables
> `/web-setup` and auto-creates environments for members; until it's on, connect via the
> browser flow.

### Cloud environments

A **cloud environment** is the saved config controlling network access, environment
variables, and the setup script that runs before each session. The same environments apply
wherever you launch a cloud session (web, terminal, mobile, Desktop, routines). Edit
`Default` or add more from the environment selector at claude.ai/code when you need a
different network level, secrets, or a setup script. Network levels range from `None`
(no internet) to `Trusted` (package registries + allowlist) to broader access.

> Setup-script gotchas: it must finish within ~5 minutes (the environment cache budget), and
> package installs need a network level that permits the registry. Add `set -x` to debug, and
> `|| true` after non-critical steps so they don't block session start.

### Start and review a task

1. **Pick a repo and branch** with the selector below the input box. You can add multiple
   repos to work across them in one session.
2. **Pick a permission mode** from the dropdown, cloud offers **Accept edits**, **Plan**,
   and **Auto** (no Manual or Bypass).
3. **Describe the task and submit.** Be specific, name the file/function, paste error
   output, describe the expected behavior. Each task gets **its own session and branch**, so
   you don't wait for one to finish before starting the next.
4. **Review the diff** (a `+42 -18` indicator opens the diff view). Select any line to leave
   an **inline comment**; comments queue and bundle with your next message, so Claude knows
   exactly where you mean.
5. **Create a PR** from the diff view (full, draft, or jump to GitHub's compose page). The
   session stays live afterward, paste CI failures or reviewer comments and ask Claude to
   address them.

### Move work between terminal and cloud

```bash
claude --cloud "Fix the flaky test in auth.spec.ts"   # kick off a cloud session from your terminal
claude --cloud "Update the API docs"                  # …and another, running in parallel
claude --teleport                                      # pull a cloud session into your terminal
claude --teleport <session-id>                         # …a specific one, with full history + branch
```

`/teleport` (inside a session) does the same as `--teleport`. The pattern Anthropic is
building toward: **plan locally, execute remotely, monitor from your phone**, then teleport
back when you're ready to review.

### Auto-fix pull requests

With the Claude GitHub App installed, enable **auto-fix** on a PR and Claude watches it in
the cloud: it resolves CI failures and addresses reviewer comments, pushes fixes (replying
under your username, labeled as Claude Code), and **asks when something is ambiguous** rather
than guessing. Connect a local session's PR to cloud auto-fix with **`/autofix-pr`**. Keep
your review standards high, auto-fix has no opinion yet on which PRs are safe to auto-merge.

### Pre-fill sessions from a link

Open claude.ai/code with the prompt, repos, and environment already selected, useful for a
"Fix with Claude" button in your issue tracker. URL-encode the values:

```text
https://claude.ai/code?prompt=Fix%20the%20login%20bug&repositories=acme/webapp
```

Parameters: `prompt` (alias `q`), `prompt_url` (fetch a longer prompt), `repositories`
(alias `repo`, comma-separated `owner/repo`), and `environment`.

---

## 11. Dependencies and housekeeping

- **Install dependencies manually** so you're sure you're on the latest version, rather than
  assuming Claude picked the right one. Then tell Claude what's installed.
- **Prune `CLAUDE.md` regularly.** Treat it like code: review it when things go wrong, delete
  anything Claude already gets right without it, and run `/doctor` to get trim suggestions.
- **`/clear` between unrelated tasks** to keep context clean, the most common cause of
  degraded output is a cluttered context window.
- **`/context`** shows what actually loaded (memory files, skills, rules). **`/doctor`** runs
  a full setup checkup. **`/mcp`** and **`/hooks`** show what's connected and configured.
- **Check `.claude/` into version control** (settings, skills, subagents, rules, commands) so
  your team shares the same setup, but keep personal/secret bits in `CLAUDE.local.md` and
  `settings.local.json`, and gitignore them.

---

## 12. Command cheat sheet

| Command / key | Does |
| --- | --- |
| `/init` | Analyze the codebase and generate `CLAUDE.md` |
| `Shift+Tab` | Cycle permission modes (Manual → acceptEdits → plan → …) |
| `Ctrl+G` | Open the current plan in your editor |
| `Esc` | Interrupt Claude, keeping context |
| `Esc Esc` / `/rewind` | Open the rewind/checkpoint menu |
| `/clear` | Reset the context window |
| `/compact [instructions]` | Summarize the conversation to free context |
| `/memory` | View/edit `CLAUDE.md` and auto memory |
| `/context` | See what's loaded into context |
| `/doctor` | Setup checkup and config diagnostics |
| `/permissions` | Manage allow/ask/deny rules |
| `/sandbox` | Toggle the sandboxed Bash tool |
| `/model` | Switch model |
| `/hooks` | Browse configured hooks |
| `/mcp` | Manage MCP servers |
| `/plugin` | Browse/install plugins |
| `/agents` | Reminder on how to create subagents (`.claude/agents/`) |
| `/code-review` | Multi-agent review of the current diff |
| `/run`, `/verify` | Launch and confirm your app works |
| `/batch <task>` | Fan a change out across subagents |
| `/loop`, `/goal` | Repeat a prompt / work toward a condition |
| `/schedule` | Create a cloud routine |
| `/web-setup` | Connect GitHub + create a cloud environment from the terminal |
| `/teleport` | Pull a cloud session into the current terminal |
| `/autofix-pr` | Connect the current PR to cloud auto-fix |
| `/rename` | Name the current session |
| `claude --continue` / `--resume` | Resume a previous session |
| `claude -p "…"` | Non-interactive (headless) run |
| `claude --permission-mode plan\|auto\|…` | Start in a specific mode |
| `claude --cloud "…"` | Start a cloud session from the terminal |
| `claude --teleport [id]` | Bring a cloud session to your terminal |
| `claude mcp add …` | Connect an MCP server |
| `@file` | Reference a file in a prompt |
| `!` `` `cmd` `` (in a skill) | Inject live command output |

---

## 13. Sources

- Course: *Claude Code: The Practical Guide* — Udemy
  <https://www.udemy.com/course/claude-code-the-practical-guide>
- Official documentation — <https://code.claude.com/docs> (Overview, Best practices, Memory,
  Skills, Subagents, Permission modes, Checkpointing, Claude Code on the web / web quickstart,
  and the docs index).

---

*These are personal learning notes. They summarize and reorganize public documentation for
my own reference and practice; they are not affiliated with or endorsed by Anthropic.*
