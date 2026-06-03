---
name: prepare-pr
description: Commit outstanding changes, open a pull request, and write its description as a structured review guide that helps humans review efficiently
model: opus
effort: high
allowed-tools: Read Grep Glob Bash
---

# Prepare PR

You are tasked with getting a change ready for review: commit any outstanding work, open a pull request, and write the PR description as a **review guide** that guides human attention to what matters. You don't replace code review — you make it faster by writing a description that tells reviewers where to look.

This is the final step of the research → design → plan → implement → review workflow.

## Initial Response

When this skill is invoked, determine the mode from `$ARGUMENTS`:

- **No arguments** → **Create mode.** Commit outstanding changes on the current branch, push, open a new PR, and write its description.
- **A PR number (e.g. `123`)** → **Update mode.** Leave git alone and rewrite the description of that existing PR. Use this when a PR was created outside this skill loop and just needs a good description.
- **A branch name** → **Create mode** targeting that branch's diff against the default branch.

If the mode is ambiguous, state your assumption and proceed; don't stall on a question you can answer from context.

## Process

### Step 1: Establish the diff and gather context

1. **Determine the default branch** (usually `main`): `git remote show origin | sed -n '/HEAD branch/s/.*: //p'` or fall back to `main`.

2. **Determine the diff source:**
   - Create mode, current branch: `git diff [default-branch]...HEAD`
   - Create mode, named branch: `git diff [default-branch]...[branch]`
   - Update mode (PR number): `gh pr diff [number]`

3. **Read the full diff.** This is the source of truth for the description.

4. **Check for supporting context (all optional):**
   - A design doc in `thoughts/shared/designs/` matching the recent dates or the branch name — read it to understand the *intent* behind the change.
   - Review metadata in `thoughts/shared/review-metadata/` matching recent dates (produced by `/implement-plan`) — use it to categorize files accurately.
   - The issue/ticket the work is tied to, if the project tracks them — link it in the PR.
   - These make the description sharper, but the review guide works from the diff alone (e.g. when preparing a PR for someone else's branch).

### Step 2: Commit outstanding changes (create mode only)

Skip this entire step in update mode.

1. **Check working tree state:** `git status --short`.

2. **If there are uncommitted changes:**
   - Review what changed and group it into a sensible commit (or a few logical commits if the work is clearly separable).
   - Optionally run the project's verification commands first (tests, lint, type-check, format) and report results — don't commit on top of a red build without flagging it.
   - Propose a commit message that follows the repository's existing convention (check `git log --oneline -10` for the style — conventional commits, ticket prefixes, etc.).
   - **Confirm the commit message with the user before committing**, then commit.
   - Never add a "Co-authored-by" / "Generated with" trailer unless the repository's own history shows that convention.

3. **If the working tree is already clean**, note that there's nothing to commit and move on.

4. **Push the branch** to the remote with upstream tracking: `git push -u origin HEAD`. Confirm before pushing if the user hasn't already signaled they want the PR created.

### Step 3: Write the review guide (the PR description)

Compose the PR description from the diff, using design docs and review metadata when present. **Categorize every changed file** as Critical (needs careful human attention), Mechanical (safe to skim or skip), or Tests (verify they test what they claim).

Use this structure for the PR body:

```markdown
## Summary
[3-5 sentences: what this change does, why, and the overall approach. Link the ticket/issue if there is one.]

## Critical Review (read carefully)

### `path/to/file.ext:lines` — [why it's critical]
- What this code does: [brief explanation]
- What to verify: [the specific thing to check]
- Risk: [what could go wrong]

### `path/to/file.ext:lines` — [why it's critical]
...

## Mechanical Changes (safe to skim)
- `path/to/file.ext` — [why it's mechanical: e.g., "type definitions mirroring schema"]
- `path/to/file.ext` — [why it's mechanical: e.g., "import reorganization"]

## Test Coverage
**Tested:**
- [Feature/behavior] — `tests/path/test.ext` ([unit/integration/e2e])

**Not tested:**
- [Feature/behavior] — [why: needs manual testing / not yet covered / trivial]

[Call out anything critical that lacks test coverage.]

## Suggested Review Order
1. [Start here because…]
2. [Then this…]
3. [Skip these unless something looks off above.]
```

**What makes a file critical:**
- Core business logic changes
- Security-sensitive code (auth, permissions, input validation, secrets)
- Data model changes (migrations, schema changes)
- External API integrations or contract changes
- Anything that deviated from the plan
- Anything not covered by tests

For long sections, wrap the lower-priority blocks (Mechanical Changes, full Test Coverage) in `<details><summary>…</summary>` so the description stays scannable.

### Step 4: Create or update the PR

**Create mode:**
1. Show the proposed title and description to the user and confirm.
2. Derive a concise PR **title** from the change (follow the repo's convention; if the branch maps to a ticket, prefix it).
3. Create the PR: `gh pr create --title "..." --body "..."` (target the default branch). Use a heredoc or `--body-file` to preserve formatting.
4. Return the PR URL.

**Update mode:**
1. Show the proposed description and confirm.
2. Update it: `gh pr edit [number] --body "..."` (use `--body-file` to preserve formatting). Leave the title alone unless it's clearly wrong — ask before changing it.
3. Return the PR URL.

## Important Guidelines

1. **Guide attention, don't find bugs.** This is not an automated code review. The description guides the *human* reviewer to the right files in the right order.
2. **Be specific about line ranges.** "Review auth.ts" is useless; "Review `auth.ts:45-67`, the new permission check" is useful.
3. **Explain *why* something is critical** — don't just flag it, say what could go wrong.
4. **Mechanical doesn't mean unimportant** — it means it follows an established pattern and doesn't need creative review.
5. **Works without metadata.** With no design doc or review metadata, analyze purely from the diff. Less precise, still useful — and exactly the situation when preparing a PR for a coworker's branch.
6. **Confirm before outward-facing or irreversible actions** — committing, pushing, creating the PR. The review guide itself is cheap; the git operations are not.
7. **Match the repo's conventions** — commit message style, PR title format, ticket linking. Read recent history rather than imposing a new style.
