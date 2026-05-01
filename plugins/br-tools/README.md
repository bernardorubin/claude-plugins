# br-tools

Bernardo's personal Claude Code toolkit — 8 slash commands and 1 skill for git workflow, PR helpers, Claude meta tasks, and external integrations.

## Installation

```
/plugin marketplace add bernardorubin/claude-plugins
/plugin install br-tools@bernardorubin-tools
/reload-plugins
```

## Commands

### Git

#### `/br-tools:git-acp`
Stage all changes, generate a concise commit message from the diff, commit (no AI co-author lines), and push to the current branch's remote. One-shot replacement for the `git add -A && git commit && git push` ritual.

#### `/br-tools:git-pull-reapply`
Bring the current branch up to date with remote while preserving local work. Handles four scenarios:
1. Clean tree + fast-forward → simple `git pull`
2. Uncommitted changes + fast-forward → stash, pull, pop
3. Clean tree + divergent branches → rebase local commits onto remote
4. Uncommitted changes + divergent branches → stash, rebase, pop

Always rebases over merging so history stays linear.

### GitHub

#### `/br-tools:pr-description`
Generates a GitHub-ready PR description from the diff and updates the PR directly via `gh`. Falls back to saving to `~/Desktop` if the GitHub update fails.

```
/br-tools:pr-description              # Auto-detect PR from current branch
/br-tools:pr-description 463          # Specific PR by number
/br-tools:pr-description <pr-url>     # Specific PR by URL
```

### Claude meta

#### `/br-tools:claude-learn`
Reviews the current session and documents valuable learnings into the right CLAUDE.md files (global, project root, or module). Helps future sessions start smarter.

```
/br-tools:claude-learn                # Review whole session
/br-tools:claude-learn <learning>     # Document a specific learning
```

#### `/br-tools:claude-modularize`
Breaks down a large, monolithic CLAUDE.md into smaller, scoped files distributed across the project's directory structure (component-specific guidelines move next to components, etc.).

### Integrations

#### `/br-tools:prd-to-jira`
Breaks down a PRD, spec, or feature document into a Jira epic with well-structured, right-sized tickets organized by work area. Backed by the bundled `prd-to-jira` skill.

```
/br-tools:prd-to-jira                          # Expects PRD pasted in conversation
/br-tools:prd-to-jira <path-or-url-or-key>     # Path, URL, or Jira ticket key
```

#### `/br-tools:save-session-to-worklog`
Logs the current session's work into a monthly worklog file on the Desktop (e.g. `~/Desktop/april-2026-happy-worklog.md`). For standups and invoicing — not git history. Auto-detects the project; multiple repos belonging to the same project share one file.

```
/br-tools:save-session-to-worklog                       # Auto-detect project
/br-tools:save-session-to-worklog --project happy       # Force project name
```

#### `/br-tools:write-slack-message`
Drafts a Slack message ready to copy-paste, with proper formatting and a business-casual tone. Asks for context if invoked with no arguments.

## Bundled skills

### `br-tools:prd-to-jira`
Same scope as the slash command but invocable via the Skill tool. Triggered automatically when the user shares a PRD, asks to "create tickets", "break this down", "make Jira tasks", etc.

## License

MIT
