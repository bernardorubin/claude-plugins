# bernardorubin-tools

A Claude Code plugin marketplace by Bernardo Rubin.

## Setup

Add the marketplace once:

```
/plugin marketplace add bernardorubin/claude-plugins
```

Then install whichever plugins you want.

## Plugins

### [`pr-review`](plugins/pr-review/) — Confidence-scored PR reviews

Runs multiple specialized review agents in parallel (security, correctness, code quality, performance), filters by confidence score, and writes a markdown report you can iterate on.

```
/plugin install pr-review@bernardorubin-tools
```

See [`plugins/pr-review/README.md`](plugins/pr-review/README.md) for full docs, modes, and examples.

---

### [`br-tools`](plugins/br-tools/) — Personal toolkit

A bundle of 8 slash commands and 1 skill covering git workflow, PR helpers, Claude meta commands, and external integrations (Slack, Jira).

```
/plugin install br-tools@bernardorubin-tools
```

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

See [`plugins/br-tools/README.md`](plugins/br-tools/README.md) for full docs on each command.

## License

MIT
