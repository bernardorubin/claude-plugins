# bernardorubin-tools

A Claude Code plugin marketplace by Bernardo Rubin. One plugin, everything bundled.

## Installation

```
/plugin marketplace add bernardorubin/claude-plugins
/plugin install br-tools@bernardorubin-tools
/reload-plugins
```

## What's inside `br-tools`

### Slash commands

| Command | Purpose |
|---------|---------|
| `/br-tools:git-acp` | Stage all, commit with auto-generated message, push |
| `/br-tools:git-pull-reapply` | Pull + preserve local work via stash/rebase |
| `/br-tools:pr-description` | Generate a PR description and update GitHub directly |
| `/br-tools:claude-learn` | Capture session learnings into CLAUDE.md files |
| `/br-tools:claude-modularize` | Split a monolithic CLAUDE.md across the project |
| `/br-tools:prd-to-jira` | Break a PRD into a Jira epic with right-sized tickets |
| `/br-tools:save-session-to-worklog` | Log session work to a monthly Desktop worklog |
| `/br-tools:write-slack-message` | Format a copy-paste-ready Slack message |

### Skills

| Skill | Purpose |
|-------|---------|
| `br-tools:pr-review` | Confidence-scored PR review with parallel specialized agents |
| `br-tools:prd-to-jira` | PRD breakdown skill (also exposed as the slash command above) |

See [`plugins/br-tools/README.md`](plugins/br-tools/README.md) for full details on every command, the PR review modes (full / lite), and the iterative review loop.

## License

MIT
