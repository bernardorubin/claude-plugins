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

### Recommended workflow

```
1. Run /pr-review              → Full review with all agents, get initial issues
2. Fix the flagged issues       → Make code changes
3. Run /pr-review --lite        → Quick re-check, resolved issues marked ✅ Fixed,
                                  new issues surfaced with fewer tokens
4. Repeat with --lite           → Until the review is clean
```

Start with a full review to get thorough coverage, then use `--lite` for subsequent passes. The lite re-checks are faster and use significantly fewer tokens since they read from the diff only and run 2 agents instead of 7. The incremental tracking works the same either way — both modes write to the same file.

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

### Core Agents (always run in full mode)

**Agent 1 — Security** | *Think like an attacker*
- Input validation and sanitization gaps
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization bypass opportunities
- Sensitive data exposure (tokens, passwords, PII in logs)
- CSRF, CORS, and header security misconfigurations
- Insecure deserialization or eval usage
- Breaking changes: searches for consumers of modified types, exports, and API signatures across the codebase and flags any that weren't updated

**Agent 2 — Correctness** | *Think like a QA engineer*
- Race conditions and concurrency issues
- Null/undefined handling and type safety holes
- Logic errors and off-by-one mistakes
- Memory leaks and resource cleanup failures
- State management bugs (stale closures, missing deps in React hooks)
- Error propagation: are errors caught, surfaced, or silently swallowed?
- Edge cases: empty arrays, zero values, undefined optional fields, boundary conditions

**Agent 3 — Code Quality & Conventions** | *Think like a senior reviewer on the team*
- TypeScript typing strictness (no `any`, proper interfaces)
- SOLID principles compliance
- DRY violations and unnecessary duplication
- Naming conventions and readability
- Project pattern adherence (reads your CLAUDE.md for project-specific conventions)
- Test coverage gaps
- Missing companion changes: sanity typegen not run, new env vars undocumented, server-to-client component conversions without loading/error states, changed shared types without updating consumers

**Agent 4 — Performance & UX** | *Think like a user on a slow connection*
- Unnecessary re-renders and missing memoization
- Inefficient queries or data fetching patterns
- Bundle size impact (distinguishes client vs server: server-only packages don't affect bundle)
- Accessibility issues (ARIA attributes, keyboard navigation, screen readers)
- Loading and error state handling gaps
- Missing cleanup (event listeners, subscriptions, timers)
- Dependency audit: if `package.json` changed, checks new packages for maintenance status, known vulnerabilities, and size; flags removed packages that might still be imported

### Specialist Agents (triggered automatically when relevant)

**Agent 5 — Silent Failure Hunter** | *Think like an oncall engineer paged at 3am*
Triggers when the diff contains `try`/`catch`, `.catch()`, `|| fallback`, or error handling changes.
- `catch` blocks that swallow errors (empty catch, catch with only `console.log`)
- Fallback values that mask failures (defaults that hide broken state)
- Missing error propagation (async functions that don't await or handle rejections)
- Logging gaps: errors caught but not logged, or logged at wrong severity
- Retry logic without backoff or max attempts
- Status checks that return success even on partial failure

**Agent 6 — Comment Accuracy** | *Think like a developer reading this code 6 months from now*
Triggers when the diff adds or modifies 5+ comment lines.
- Comments that contradict the code they describe
- Stale comments referencing removed/renamed variables, functions, or logic
- TODO/FIXME/HACK comments without context or tracking
- Over-commenting (restating what the code clearly does)
- Under-commenting (complex logic with no explanation)
- JSDoc/docstring parameter mismatches (wrong types, missing params, extra params)

**Agent 7 — Type Design** | *Think like a library author*
Triggers when the diff introduces new `type` or `interface` definitions.
- Types that allow invalid states (e.g., `status: string` instead of a union)
- Missing `readonly` modifiers on immutable data
- Overly broad types (`any`, `object`, `Record<string, unknown>`) where narrower types are possible
- Discriminated unions that should be used but aren't
- Types that don't enforce their business rules (e.g., email as `string` vs branded type)
- Exported types that leak implementation details

### Lite Mode Agents

In `--lite` mode, the 4 core agents are consolidated into 2 for speed:

**Agent A — Security & Correctness**: Combines Agent 1 + Agent 2 focus areas, reviews from diff only
**Agent B — Quality & Performance**: Combines Agent 3 + Agent 4 focus areas, reviews from diff only

Specialist agents (5, 6, 7) still trigger in lite mode when relevant.

## How It Compares

`/pr-review` vs Anthropic's built-in `review-pr` toolkit:

![Comparison](plugins/pr-review/comparison.png)

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
