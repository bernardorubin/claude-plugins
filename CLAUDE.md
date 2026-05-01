# claude-plugins (bernardorubin-tools marketplace)

This repo is a Claude Code **plugin marketplace**. It is not application code — there is no build, no test runner, and no package manager. Everything here is JSON manifests + markdown command/skill definitions consumed by the Claude Code harness.

## Repo layout

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json         # marketplace manifest — lists every plugin
├── plugins/
│   └── br-tools/                # the single bundled plugin (see below)
│       ├── .claude-plugin/
│       │   └── plugin.json      # plugin manifest
│       ├── commands/            # slash commands as .md files
│       ├── skills/              # skill folders, each with SKILL.md
│       ├── README.md            # user-facing plugin docs
│       └── comparison.png       # asset referenced by README
└── README.md                    # marketplace overview, install instructions
```

## How everything wires together

- **Marketplace name**: `bernardorubin-tools` (set in `.claude-plugin/marketplace.json`)
- **GitHub identifier**: `bernardorubin/claude-plugins` (used in `/plugin marketplace add`)
- **Single plugin**: `br-tools` — bundles 8 commands + 2 skills (`pr-review`, `prd-to-jira`)
- **Install path** (after `/plugin install`): `~/.claude/plugins/cache/bernardorubin-tools/br-tools/<version>/`

When users update the marketplace and reinstall, the harness pulls from `main` of this repo via the `git-subdir` source defined in `marketplace.json`.

## Adding a new command

1. Create `plugins/br-tools/commands/<name>.md` — first line `# Title`, then a description and instructions. No frontmatter required for commands.
2. Add a row to the command table in `plugins/br-tools/README.md`.
3. Add a row to the command table in the root `README.md`.
4. Commit + push. Users run `/plugin marketplace update bernardorubin-tools` then `/plugin install br-tools@bernardorubin-tools` to refresh.

The command becomes invocable as `/br-tools:<name>` after install + `/reload-plugins`.

## Adding a new skill

1. Create `plugins/br-tools/skills/<name>/SKILL.md` with frontmatter:
   ```
   ---
   name: <name>
   description: <one-line description that drives auto-trigger>
   ---
   ```
2. Optionally add `evals/`, `references/`, etc. as siblings of `SKILL.md`.
3. Document in `plugins/br-tools/README.md` under "Skills" and in the root README's skills table.

The skill becomes available as `br-tools:<name>` for the Skill tool.

## Conventions

- **Naming**: keep command/skill names short and descriptive. They're prefixed with `br-tools:` automatically — no need for extra prefixes.
- **No AI co-author lines** in commit messages (the user handles git operations themselves; never run `git commit` without being asked).
- **README is the source of truth** for what each command/skill does — keep it in sync when commands change.
- **Single source of truth**: command/skill files live ONLY here. The user's `~/.claude/commands/` and `~/.claude/skills/` should not contain copies of anything bundled in `br-tools` (avoids drift).

## Versioning

`plugin.json` and `marketplace.json` both have a `version` field — keep them in sync. Bump on meaningful changes (new command, new skill, behavior change). Patch bumps for typo/doc-only changes are optional.

## Publishing flow

1. Make changes, validate JSON files parse:
   ```bash
   python3 -c "import json; json.load(open('.claude-plugin/marketplace.json'))"
   python3 -c "import json; json.load(open('plugins/br-tools/.claude-plugin/plugin.json'))"
   ```
2. Commit + push to `main`.
3. On any machine that already has the marketplace: `/plugin marketplace update bernardorubin-tools` → `/plugin install br-tools@bernardorubin-tools` to pull updates.

There is no app store, approval process, or release pipeline — pushing to `main` is publishing.
