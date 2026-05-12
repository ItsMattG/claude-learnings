# Claude Code Onboarding Guide

> A practical handover for someone moving from the Claude VS Code extension to the full Claude Code CLI workflow — plugins, skills, memory, hooks, subagents, and end-to-end feature flow.

---

## Simplified Version — "Just Get Started"

Claude Code on the CLI is way more powerful than the VS Code extension because of **plugins**, **skills**, **memory**, and **hooks**. Think of it as the difference between asking Claude questions vs giving Claude a full toolbox + long-term memory.

### The 4 things to set up on Day 1

1. **Install Claude Code CLI**
   ```bash
   npm i -g @anthropic-ai/claude-code
   cd ~/your-project
   claude
   ```

2. **Add a `CLAUDE.md` at the project root** — the rulebook Claude reads on every conversation. Run `/init` to scaffold one, then tighten it: stack, conventions, "always do X / never do Y."

3. **Install the official plugin marketplace** (free, made by Anthropic). Inside Claude:
   ```
   /plugin
   ```
   Enable these 5 — they unlock most of the workflow:
   - `superpowers` — brainstorm → plan → execute → review skill set
   - `frontend-design` — UI critique, polish, animate
   - `context7` — live library docs so Claude doesn't hallucinate APIs
   - `typescript-lsp` — Claude navigates code like an IDE
   - `playwright` (web E2E) or `vercel` (if you deploy there)

4. **Let memory build itself**. Claude auto-saves preferences, project facts, and feedback to `~/.claude/projects/<your-project>/memory/`. After 5–10 sessions it remembers your style, your stack, and prior decisions across conversations.

### The Day-1 workflow (5 commands)

| Stage | Type this | What it does |
|---|---|---|
| 1. Idea | `/brainstorming` | Challenges the idea before coding |
| 2. Plan | `/writing-plans` | Numbered plan saved to `docs/plans/` |
| 3. Execute | `/executing-plans` | Runs the plan task-by-task |
| 4. Review | `/requesting-code-review` | Spawns code-reviewer subagent |
| 5. Ship | `/finishing-a-development-branch` | Push, PR, summary |

All five come free with the `superpowers` plugin. **That alone is 80% of the value.**

### Three habits that matter most

- **`/clear` between unrelated tasks** — context is the most precious resource.
- **Use git worktrees** for parallel features (`git worktree add ~/wt/feat-x -b feature/x develop`) so Claude can't break your main branch.
- **Correct Claude when it's wrong** — say "no, do X because Y." That correction gets saved as a `feedback` memory and Claude won't make the same mistake again.

---

## Thorough Detailed Guide

### 0. Mental model — why CLI > VS Code extension

The VS Code extension is "Claude in a chat box." The CLI version is **Claude as a configurable agent** with:

- **Plugins** — installable bundles of skills/agents/hooks
- **Skills** — packaged procedural knowledge Claude auto-invokes
- **Subagents** — specialist Claudes spawned for focused jobs
- **Hooks** — shell scripts the harness runs on tool events
- **Slash commands** — your own `/commands` that expand into prompts
- **Memory** — persistent files across sessions
- **MCP servers** — give Claude real tools (Chrome DevTools, Supabase, Vercel)

It's the difference between "AI typing buddy" and "AI engineer that learns your codebase."

### 1. Install & first-run

```bash
npm i -g @anthropic-ai/claude-code
cd ~/dev/your-project
claude
# Inside Claude:
/init
```

`/init` writes a starter `CLAUDE.md`. Tighten it: stack, conventions, things to never do. A good one is short — 1-page cheatsheet, DO/DON'T tables, links to feature-area docs.

#### Settings: `.claude/settings.json` (project) + `~/.claude/settings.json` (global)

Useful keys:
```json
{
  "model": "opus",
  "effortLevel": "xhigh",
  "permissions": { "defaultMode": "bypassPermissions" },
  "enabledPlugins": { },
  "env": { "ENABLE_LSP_TOOL": "1" }
}
```

`bypassPermissions` skips most tool prompts — only safe if you trust your hooks.

### 2. Plugins — the highest-leverage install

Run `/plugin` inside Claude. Recommended priority:

| Plugin | Purpose |
|---|---|
| **superpowers** | Process: brainstorming, planning, execution, TDD, debugging, review |
| **frontend-design** | UI quality skills (`polish`, `critique`, `animate`, `colorize`) |
| **context7** | Live library docs (Next, tRPC, Drizzle, Zod, etc.) |
| **typescript-lsp** | Real LSP — `goToDefinition`, `findReferences`, `hover` |
| **playwright** | Browser automation MCP for E2E + UI verification |
| **vercel** | Vercel CLI + verification skills |
| **supabase** | Supabase MCP + postgres best-practices |

**Start with `superpowers` + `context7` + `typescript-lsp`.** Add the others as needed.

```
/plugin                       # interactive UI
/plugin install <name>
/plugin disable <name>
```

### 3. Skills — what they are and how they fire

A skill is a markdown file with frontmatter:

```markdown
---
name: brainstorming
description: Use when starting any new feature idea before planning
---
# Brainstorming
1. Ask: who is this for?
2. Find the smallest version that delivers value.
...
```

Claude matches your prompt against each skill's `description` and invokes the relevant one automatically.

#### Two sources

1. **From plugins** — `superpowers:brainstorming`, `frontend-design:polish`, etc.
2. **Custom in your project** — `.claude/skills/<name>/SKILL.md`

#### Custom skills worth copying

- `/new-component` — scaffolds a React component matching conventions
- `/new-router` — scaffolds a tRPC router
- `/new-e2e-test` — scaffolds a Playwright spec
- `/ship` — finishes a branch end-to-end (push, PR, summary)
- `/wrap-up` — end-of-session retro: commits, learnings, memory updates
- `/verify-preview <pr#>` — Vercel preview verification with Chrome DevTools
- `/scan-todos`, `/debt-scan`, `/audit-deps` — codebase health passes

**Create one:** make `.claude/skills/<name>/SKILL.md` with frontmatter and clear `description`. Or run `/skill-create` — it analyses recent work and writes the skill file for you.

### 4. The end-to-end flow

```
Idea → Brainstorm → Requirements → Plan → Worktree → Execute (TDD) →
Verify (Playwright/preview) → Code review → Ship (PR) → Wrap-up (memory)
```

#### Stage 1 — Brainstorm
`/superpowers:brainstorming` — Claude refuses to plan or code. Asks clarifying questions, surfaces unknowns, pushes back on weak ideas.

Second opinion: `/gemini-brainstorm` offloads research to Gemini and feeds the result back. Cross-model adversarial thinking is cheap insurance.

#### Stage 2 — Requirements / Design
`/requirements-intake` — structured Q&A producing a tight requirements doc.

UI work: design context lives in `CLAUDE.md`. A 200-word section on brand personality + visual references is enough for Claude to match your style.

Visual exploration: `/frontend-design:frontend-design` produces distinctive, production-ready UI directions.

#### Stage 3 — Plan
`/superpowers:writing-plans` — saves a numbered markdown plan to `docs/plans/YYYY-MM-DD-<slug>.md`. Each step has acceptance criteria. This is your contract with Claude.

Adversarially review the plan before executing: `/review-plan`.

Epic-sized work: `/plan-to-epic` splits a plan into GitHub issues with sub-tasks.

#### Stage 4 — Worktree (critical)
Never work feature changes on `main` or `develop` directly.

```bash
git worktree add ~/worktrees/your-project/<feat-name> -b feature/<name> develop
cp .env.local ~/worktrees/your-project/<feat-name>/.env.local
cd ~/worktrees/your-project/<feat-name>
claude
```

Worktrees let you run 3–4 Claude sessions in parallel on different branches with zero cross-contamination.

#### Stage 5 — Execute the plan
`/superpowers:executing-plans` walks the plan task-by-task: mark in-progress → read with LSP → write change → typecheck on changed files → mark complete.

- TDD-required features: `/superpowers:test-driven-development` — refuses impl until a failing test exists.
- Parallel work: `/superpowers:subagent-driven-development` — Claude orchestrates, subagents do focused tasks.

#### Stage 6 — Verify
This is where most people ship bugs. Don't.

- **Unit:** `pnpm test:unit` (Vitest)
- **Integration:** real DB, no mocks
- **E2E:** `pnpm test:e2e` (Playwright)
- **Visual:** `/verify-preview <pr#>` opens the Vercel preview via Chrome DevTools MCP, navigates key routes, screenshots, reports console errors
- **Completion gate:** `/superpowers:verification-before-completion`

#### Stage 7 — Code review
`/superpowers:requesting-code-review` or the `code-reviewer` subagent.

Specialist reviewers (`.claude/agents/*.md`):
- `code-reviewer.md` — project conventions
- `database-reviewer.md` — Drizzle/Postgres perf & correctness
- `security-reviewer.md` — OWASP top-10
- `build-error-resolver.md` — TS/build errors
- `test-writer.md` — Vitest/Playwright tests

Each runs in fresh context with only the tools it needs — main conversation stays clean.

Adversarial review of the PR: `/security-review` or `/review` (from the `code-review` plugin).

#### Stage 8 — Ship
`/ship` (custom) or `/superpowers:finishing-a-development-branch`: push → open PR → summarise commits → verify preview → present options.

Tip: save a `feedback` memory like "never auto-merge — always present options" so Claude always stops and asks.

#### Stage 9 — Wrap-up
`/wrap-up` — end-of-session retro. Commits anything outstanding, summarises changes, prompts memory updates, surfaces "what should be saved for next time?" This is what makes Claude smarter across sessions.

Breaking mid-task: `/document-continue` saves `PROGRESS-<task>.md` so you can `/clear` and resume cleanly later.

### 5. Memory — the unfair advantage

`~/.claude/projects/<project-hash>/memory/` holds markdown files Claude writes on its own. Four kinds:

| Type | Example |
|---|---|
| `user` | "Senior eng, 10 yr Go, new to React — frame frontend in backend analogues" |
| `feedback` | "Don't auto-merge PRs — always present options" |
| `project` | "Auth middleware rewrite is compliance-driven, not cleanup" |
| `reference` | "Pipeline bugs tracked in Linear project INGEST" |

Claude saves these automatically when you correct it, confirm something non-obvious worked, or teach it a fact. You can also force a save: "remember that the demo password is X."

`MEMORY.md` at the top of that folder is an index — auto-loaded into every session. Keep it under 200 lines.

### 6. Hooks — your safety rails

Hooks are shell scripts the harness runs on tool events. The most valuable ones:

| Hook | Event | Effect |
|---|---|---|
| `block-work-on-develop.sh` | PreToolUse(Edit/Write) | Forces worktree (no `src/` edits on develop) |
| `block-destructive.sh` | PreToolUse(Bash) | Blocks `rm -rf ~/...`, force-push to main |
| `check-anti-patterns.sh` | PostToolUse(Edit/Write) | Flags `any`, `Record<string, unknown>`, console.log |
| `typecheck-changed.sh` | PostToolUse(Edit/Write) | Runs `tsc` on just changed files |
| `detect-unscoped-queries.sh` | PostToolUse(Edit) | Catches DB queries missing user-scope (security) |
| `validate-commit-msg.sh` | PreToolUse(Bash) | Enforces `<type>: <desc>` format |
| `session-start.sh` | SessionStart | Prints env status, branch, ready tasks |
| `notify-on-stop.sh` | Stop | macOS notification when Claude finishes |

Configure in `.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "~/.claude/hooks/block-destructive.sh" }]
      }
    ]
  }
}
```

Use the `/update-config` skill to set hooks without hand-editing JSON.

### 7. Subagents — when to spawn one

Spawn when:
- You need **isolated context** (research, large reads) to keep the main conversation small
- You want **specialist judgement** (security, db, design)
- You can **parallelise** 2+ independent investigations

Don't spawn when:
- The task is small and you have the context already
- You'd "delegate understanding" — i.e. the agent has to re-derive what you already know

Slash command vs. Agent: a slash command is a prompt template. An agent is a sub-Claude with its own context, tools, and system prompt. Both are useful — agents for parallelism/isolation, slash commands for repeatable workflows.

### 8. MCP servers — Claude with real tools

Worth enabling:
- **chrome-devtools** — open pages, click, screenshot, read console, Lighthouse
- **supabase** — query prod DB, run RPCs, manage auth
- **vercel** — list deployments, get logs, manage env vars

Install via `/plugin` or add manually to `.claude/mcp-profiles.json`. Vercel-hosted apps: chrome-devtools + vercel together make preview verification a one-command operation.

### 9. Choosing the model — Opus vs Sonnet vs Haiku

Switch mid-conversation with `/model opus` or `/model sonnet`. Or set the default in `~/.claude/settings.json`.

| Model | Strengths | Use for |
|---|---|---|
| **Opus 4.7** | Deepest reasoning, best judgement, slowest, most expensive | Planning, architecture, debugging hard bugs, code review of nuanced changes, security review, anything where being wrong is costly |
| **Sonnet 4.6** | Fast, very capable, much cheaper | Executing well-defined plans, scaffolding, refactoring, test writing, bulk edits, parallel subagent dispatches |
| **Haiku 4.5** | Fastest, cheapest | Quick lookups, formatting, one-off questions, CI/watch-mode helpers, lightweight verification |

#### Practical routing

| Stage | Model | Why |
|---|---|---|
| Brainstorming / requirements | **Opus** | Pushback quality matters most here |
| Plan writing & review | **Opus** | The plan is the cheapest thing to fix; reasoning depth pays off |
| Executing a clear plan | **Sonnet** | Path is known, speed > depth, much cheaper for long sessions |
| TDD red-green cycles | **Sonnet** | Mechanical loop, no novel reasoning needed |
| Hard debugging (root cause unclear) | **Opus** | Hypothesis generation needs the reasoning depth |
| Easy bug triage / typo fixes | **Sonnet** or **Haiku** | Don't burn Opus on trivial work |
| Code review (security / DB / architecture) | **Opus** | Catching subtle issues is the whole point |
| Code review (style / convention) | **Sonnet** | Mechanical pattern-matching |
| Scaffolding (new component, new router) | **Sonnet** | Template-shaped work |
| Parallel subagent dispatches | **Sonnet** | Cheaper per agent + each agent has narrow scope |
| Long parallel exploration | **Haiku** or **Sonnet** | Cheap reads, return findings to Opus main thread |
| One-off questions / lookups | **Haiku** | Sub-second feel, near-zero cost |

#### Rules of thumb

- **Default to Opus for the main conversation** and let it spawn Sonnet subagents for execution. The main thread does the thinking; subagents do the typing.
- **Switch to Sonnet for long execution sessions** where you're just running a plan and the steps are clear. You'll burn far less budget and finish faster.
- **Never use Haiku for plan creation or code review.** It's an excellent assistant model but not a judgement model.
- **Use `effortLevel: "xhigh"` on Opus** for deeper thinking (slower output but markedly better reasoning on hard problems).
- **`/fast` toggles fast mode on Opus 4.6** — faster output without dropping to a smaller model. Useful when you're iterating quickly.

#### Cost-aware patterns

- **Brainstorm in Opus, execute in Sonnet:** start Opus, do brainstorm + plan, then `/model sonnet` for `/executing-plans`. Switch back to Opus for review.
- **Subagent-driven dev with Sonnet workers:** Opus orchestrator dispatches Sonnet subagents for each plan step. Orchestrator gets specialist judgement when stitching results back together.
- **Verification with Sonnet/Haiku:** `/verify-preview` doesn't need Opus — Sonnet drives Chrome DevTools fine.

### 10. Slash commands cheat-sheet

```
/init                          # bootstrap CLAUDE.md
/clear                         # wipe context between tasks
/plugin                        # manage plugins
/model opus|sonnet|haiku       # switch models mid-conversation
/fast                          # toggle fast mode (Opus 4.6 only)
/skill-create                  # generate a new skill from recent work

# Workflow
/brainstorming                 # before any new idea
/requirements-intake           # structured Q&A
/writing-plans                 # plan to file
/review-plan                   # adversarial plan review
/executing-plans               # run the plan
/tdd-workflow                  # test-first execution
/verify-preview <pr#>          # visual verification
/requesting-code-review        # spawn code reviewer
/ship                          # push + PR + summary
/wrap-up                       # end-of-session retro
/document-continue             # save state before /clear

# Scaffolds
/new-component
/new-router
/new-e2e-test

# Quality
/scan-todos
/debt-scan
/audit-deps
/security-review
/review-claudemd               # audit CLAUDE.md files
/rules-doctor                  # find broken rules

# Design (after frontend-design plugin)
/critique /polish /animate /simplify /clarify /bolder /quieter
```

### 11. A realistic session, start to finish

Adding "comments on a post" to a blog:

```
$ cd ~/dev/his-blog
$ claude

> I want to add comments to blog posts.

[Claude invokes /brainstorming]
Claude: Before we plan — who can comment? Anonymous, or logged-in only?
        What about moderation, spam, edit/delete windows? Threaded or flat?

> Logged-in only, flat, no edit, soft-delete by owner.

[Claude invokes /writing-plans, saves docs/plans/2026-05-12-comments.md]

> Looks good. Worktree it and execute.

[Claude runs: git worktree add ... && cd ... && /executing-plans]
[For each task: writes test → writes code → typecheck → commit]

> Verify.

[Claude runs vitest, then /verify-preview after push]

> Review and ship.

[/requesting-code-review → code-reviewer subagent → fixes nits]
[/ship → push, open PR, summarise, ask before merge]

> Merge it.

[Merge, delete worktree branch]

> /wrap-up

[Claude commits memory updates, summarises learnings, suggests one
 instinct to add to .claude/instincts/]
```

That whole loop, hands-mostly-off, in a single Claude session.

### 12. Pitfalls to avoid early on

- **Don't put everything in one big CLAUDE.md.** Scope it: project root has stack + global rules; `src/server/CLAUDE.md` has server rules. Claude reads the closest one to the file it's editing.
- **Don't disable hooks to "make Claude faster."** Hooks catch bugs before commit; the seconds you save will cost you hours debugging.
- **Don't accept the first plan.** Run `/review-plan`. The plan is the cheapest thing to fix.
- **Don't skip `/clear`.** Context bloat is the #1 killer of output quality.
- **Don't trust Claude's memory of APIs.** Use `context7` — it pulls live docs. Stops 80% of "this function doesn't exist" errors.
- **Don't use Opus for everything.** Sonnet for execution, Opus for judgement.

### 13. Where to go next

- Claude Code docs: https://docs.claude.com/en/docs/claude-code
- Official plugin marketplace — browseable via `/plugin`
- Read the `superpowers` plugin skills at `~/.claude/plugins/cache/claude-plugins-official/superpowers/<version>/` for inspiration
- After 1 week of use, open your own `MEMORY.md` and prune what Claude saved
