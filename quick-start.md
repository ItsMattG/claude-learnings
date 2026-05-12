# Quick Start — Claude Code in 5 Minutes

> The simplified version. If you've only used Claude in VS Code, start here, build for a day, then read [`full-guide.md`](./full-guide.md).

Claude Code on the CLI is way more powerful than the VS Code extension because of **plugins**, **skills**, **memory**, and **hooks**. Think of it as the difference between asking Claude questions vs giving Claude a full toolbox + long-term memory.

## The 4 things to set up on Day 1

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

## The Day-1 workflow (5 commands)

| Stage | Type this | What it does |
|---|---|---|
| 1. Idea | `/brainstorming` | Challenges the idea before coding |
| 2. Plan | `/writing-plans` | Numbered plan saved to `docs/plans/` |
| 3. Execute | `/executing-plans` | Runs the plan task-by-task |
| 4. Review | `/requesting-code-review` | Spawns code-reviewer subagent |
| 5. Ship | `/finishing-a-development-branch` | Push, PR, summary |

All five come free with the `superpowers` plugin. **That alone is 80% of the value.**

## Three habits that matter most

- **`/clear` between unrelated tasks** — context is the most precious resource.
- **Use git worktrees** for parallel features (`git worktree add ~/wt/feat-x -b feature/x develop`) so Claude can't break your main branch.
- **Correct Claude when it's wrong** — say "no, do X because Y." That correction gets saved as a `feedback` memory and Claude won't make the same mistake again.

## When you're ready for more

Read [`full-guide.md`](./full-guide.md) — covers skills, hooks, subagents, MCP servers, memory in depth, the full feature-dev flow, and **when to use Opus vs Sonnet vs Haiku**.
