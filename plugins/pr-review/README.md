# pr-review

A Claude Code plugin for thorough, confidence-scored PR reviews with parallel specialized agents and incremental tracking.

## What It Does

Runs multiple focused review agents in parallel, each examining your PR from a different angle (security, correctness, code quality, performance). Findings are scored on a 0-100 confidence scale, and only issues scoring 80+ are surfaced — cutting noise while catching real problems.

Results are saved to a markdown file so you can share them, reference them later, or track progress as you fix issues.

## Scope

**General-purpose, optimized for TypeScript/JavaScript projects.** Works with any language but includes specialized checks for React, Next.js, and TypeScript codebases. Frontend-specific checks (re-renders, bundle size, accessibility) are only flagged when relevant to the changed files.

## Installation

Add the marketplace and install:

```
/plugin marketplace add bernardorubin/claude-plugins
/plugin install pr-review@bernardorubin-tools
```

## Usage

```
/pr-review 463                          # Full review of PR #463, saved to ~/Desktop
/pr-review 463 --lite                   # Lightweight review (fewer agents, diff-only reads)
/pr-review 463 --inline                 # Full review, output in conversation only
/pr-review 463 --output ~/reviews       # Full review, custom output directory
/pr-review 463 --lite --inline          # Combine flags freely
/pr-review                              # Auto-detect PR from current branch
```

### Flags

| Flag | What it does | Default |
|------|-------------|---------|
| `--lite` | Uses 2 sonnet agents instead of 4+, reads diff only instead of full files, skips code snippets in output | Off (full review) |
| `--inline` | Outputs review to conversation, skips file creation | Off (saves to file) |
| `--output <path>` | Sets custom directory for review file | `~/Desktop` |

### Model Behavior

The skill uses whatever model you're currently running in Claude Code. It does not override your model choice. In `--lite` mode, subagents specifically use Sonnet for token efficiency, but the main orchestration stays on your active model.

## Full vs Lite Mode

| | Full (default) | Lite (`--lite`) |
|---|---|---|
| **Core agents** | 4 specialized (security, correctness, quality, performance) | 2 combined (security+correctness, quality+performance) |
| **Specialist agents** | Up to 3 additional (silent failures, comments, types) when triggered | Same triggers apply |
| **File reading** | Every changed file read in full | Diff only, selective file reads |
| **Code snippets** | Before/after fix suggestions included | Descriptions only |
| **Subagent model** | Your active model | Sonnet |
| **Direct review threshold** | ≤3 files / ≤150 lines | ≤8 files / ≤500 lines |
| **Best for** | Final reviews, security-sensitive changes, large PRs | Day-to-day PRs, quick checks, iterating on fixes |

## The Review Loop

This skill is designed for iterative use, not just one-shot reviews.

### How it works

```
1. Run /pr-review          → Initial review, issues identified
2. Fix the flagged issues   → Make code changes
3. Run /pr-review again     → Resolved issues marked ✅ Fixed (strikethrough),
                              new issues from your fixes surfaced
4. Repeat                   → Until the review is clean
```

### What happens on re-runs

When the skill detects a prior review file (from the same PR, same day):

- **Resolved issues** get ~~strikethrough~~ with a ✅ Fixed badge — they stay visible for history but are excluded from issue counts
- **Still-open issues** remain unchanged
- **New issues** are appended to the appropriate severity section
- **Issue counts and risk level** are recalculated based on open issues only
- **A revision entry** is added to the log at the bottom of the file

You can mix modes across runs. For example: run a full review first, then use `--lite` for quick re-checks as you iterate. Both write to the same file.

### Example revision log

| # | Time | Mode | Summary |
|---|------|------|---------|
| 1 | 2026-03-15 10:30 | Full | Initial review — 2 critical, 4 improvements |
| 2 | 2026-03-15 11:45 | Lite | Re-review — 2 fixed, 1 new suggestion |
| 3 | 2026-03-15 14:00 | Lite | Final check — all clear |

## Review Agents

### Core Agents (always run)

| Agent | Perspective | Focus Areas |
|---|---|---|
| **Security** | Think like an attacker | Injection, auth, data exposure, CSRF, breaking changes |
| **Correctness** | Think like a QA engineer | Race conditions, null handling, logic errors, edge cases |
| **Code Quality** | Think like a senior reviewer | TypeScript typing, SOLID, DRY, conventions, missing companion changes |
| **Performance & UX** | Think like a user on slow connection | Re-renders, bundle size, a11y, dependency audit |

### Specialist Agents (triggered automatically when relevant)

| Agent | Triggers when | Focus |
|---|---|---|
| **Silent Failure Hunter** | Diff contains try/catch, .catch(), fallback values | Error swallowing, missing propagation, logging gaps |
| **Comment Accuracy** | Diff adds/modifies 5+ comment lines | Stale comments, contradictions, JSDoc mismatches |
| **Type Design** | Diff introduces new type/interface definitions | Invalid state types, missing readonly, overly broad types |

## Confidence Scoring

Every finding is scored 0-100:

- **0-49**: Likely false positive or pre-existing, filtered out
- **50-79**: Might be an issue but below threshold, filtered out
- **80-100**: High confidence, included in review

If 2+ agents independently flag the same issue, its severity gets boosted one tier (suggestion → improvement → critical). This cross-agent agreement is a strong signal.

## Output Format

Reviews are saved as markdown files: `pr-review-{PR_NUMBER}-{YYYY-MM-DD}.md`

The file includes:
- Risk assessment (🟢 LOW / 🟡 MEDIUM / 🔴 HIGH / ⛔ CRITICAL)
- Issues grouped by severity with location, confidence score, and impact
- Before/after code snippets (full mode only)
- Breaking changes and dependency notes
- Good practices observed
- Issues indexed by file
- Revision history

## License

MIT
