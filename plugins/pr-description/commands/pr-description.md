# Generate PR Description

Generate a GitHub-ready PR description and update the PR directly on GitHub. Falls back to saving to Desktop if the GitHub update fails.

## Arguments

`$ARGUMENTS` - Optional PR number or URL (e.g., `463` or `https://github.com/org/repo/pull/463`)

## Instructions

### Step 1: Get PR Information

Try these in order until one works:

1. **If argument provided**: Run `gh pr view $ARGUMENTS --json title,number,url,commits,files,baseRefName,headRefName`
2. **If no argument**: Run `gh pr view --json title,number,url,commits,files,baseRefName,headRefName` (auto-detects current branch's PR)
3. **If both fail**: Fall back to `git log` and `git diff --stat` against the base branch (usually `main` or `master`)

Save the PR number and URL for later steps.

### Step 2: Analyze Changes

From the PR data or git diff:
- Review all commits to understand the scope of work
- Analyze files changed to identify key areas
- Group related changes into logical categories

### Step 3: Generate PR Description

Create the description body (no H1 title — GitHub already shows the PR title). Use this structure:
- `## Summary` - Bullet points summarizing what changed
- `## What's New` - Tables for files changed, features added
- `## How to Test` - Step-by-step instructions
- `## Test Plan` - Checkbox items `- [ ]` for testing

Write the description to a temp file at `/tmp/pr-description.md`.

### Step 4: Update PR on GitHub

Try to update the PR description directly on GitHub:

```bash
gh pr edit <PR_NUMBER> --body-file /tmp/pr-description.md
```

**If successful**: Tell the user the PR description was updated and show the PR URL.

**If failed** (e.g., no PR number, auth issues, not a GitHub repo): Fall back to saving at `~/Desktop/pr-description.md` and tell the user it couldn't update GitHub directly, so the file was saved to Desktop instead.

Clean up the temp file after either outcome.

## Format Requirements

- Use `##` for main sections, `###` for subsections
- Use tables for structured data (files, components, settings)
- Use bullet points (-) for features/changes
- Use numbered lists for step-by-step instructions
- Use checkboxes (- [ ]) for test plan items
- Include code blocks with proper language identifiers
- Link related PRs if work spans multiple repos
- Include file paths in backticks
- Do NOT include any AI/Claude attribution or signatures
- Do NOT wrap output in code fences - write raw markdown
