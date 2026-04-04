---
name: review-changes
description: Generate a structured review guide for code changes to help humans review efficiently
model: opus
effort: medium
allowed-tools: Read Grep Glob Bash
---

# Review Changes

You are tasked with generating a structured review guide that helps humans efficiently review code changes. You guide attention to what matters — you don't replace code review, you make it faster.

## Initial Response

When this command is invoked:

1. **Determine the diff source from $ARGUMENTS:**
   - If a PR number is provided: fetch the PR diff using `gh pr diff [number]`
   - If a branch name is provided: diff against main with `git diff main...[branch]`
   - If no args: diff current branch against main with `git diff main...HEAD`

2. **Check for supporting context:**
   - Look for a design doc in `thoughts/shared/designs/` matching recent dates or branch name
   - Look for review metadata in `thoughts/shared/review-metadata/` matching recent dates
   - These are optional — the review guide works from the diff alone

## Process

### Step 1: Analyze the Diff

1. **Read the full diff**
2. **If a design doc exists**, read it to understand the intent behind the changes
3. **If review metadata exists** (from `/implement-plan`), use it to categorize files
4. **Categorize every changed file** into one of:
   - **Critical** — Needs careful human attention
   - **Mechanical** — Safe to skim or skip
   - **Tests** — Verify they test what they claim

### Step 2: Generate the Review Guide

Present the review guide directly to the user:

```markdown
# Review Guide: [Brief Description]

## Summary
[3-5 sentences: what this change does, why, and the overall approach]

## Critical Review (read carefully)
[Files and specific line ranges that need human attention]

### [filename:lines] — [why it's critical]
- What this code does: [brief explanation]
- What to verify: [specific thing to check]
- Risk: [what could go wrong]

### [filename:lines] — [why it's critical]
...

**Categories that make code critical:**
- Core business logic changes
- Security-sensitive code (auth, permissions, input validation, secrets)
- Data model changes (migrations, schema changes)
- External API integrations or contract changes
- Anything that deviated from the plan
- Anything not covered by tests

## Mechanical Changes (safe to skim)
[Files that are boilerplate, generated, or follow established patterns]

- `path/to/file.ext` — [why it's mechanical: e.g., "type definitions mirroring schema"]
- `path/to/file.ext` — [why it's mechanical: e.g., "import reorganization"]

## Test Coverage
### What's tested:
- [Feature/behavior] — `tests/path/test.ext` ([unit/integration/e2e])

### What's NOT tested:
- [Feature/behavior] — [why: needs manual testing / not yet covered / trivial]

**Highlight:** [Call out anything critical that lacks test coverage]

## Patterns to Verify
[Quick spot-checks — not full reads, just "does this look consistent?"]

- "Changes in `path/to/file.ext` follow the pattern established in `path/to/reference.ext`"
- "New endpoint follows the same auth middleware chain as existing endpoints"

## Suggested Review Order
1. [Start with this file because...]
2. [Then this file...]
3. [Skip these files unless something looks off in step 1-2]
```

### Step 3: Offer to Post to PR

After presenting the review guide:

```
Would you like me to post this review guide as a comment on the PR?
- If yes, I'll format it with collapsible sections and post via `gh`
- If no, the guide above is yours to reference locally
```

If the user says yes:
1. Format the guide with `<details><summary>` collapsible sections for each major section
2. Post using `gh pr comment [number] --body "..."`
3. If no PR number was provided, ask for one or offer to create the PR first

## Important Guidelines

1. **Guide attention, don't find bugs** — This is not an automated code review. The existing `/review` command does that. This guides the HUMAN reviewer.
2. **Be specific about line ranges** — "Review auth.ts" is useless. "Review auth.ts:45-67, the new permission check" is useful.
3. **Explain why something is critical** — Don't just flag it, explain what could go wrong.
4. **Mechanical doesn't mean unimportant** — It means it follows established patterns and doesn't need creative review.
5. **Works without metadata** — If no design doc or review metadata exists, analyze purely from the diff. The guide is less precise but still useful.
6. **Works for coworker PRs** — When reviewing someone else's PR, you won't have design docs. That's fine — work from the diff.
