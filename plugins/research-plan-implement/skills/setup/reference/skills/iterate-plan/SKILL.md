---
name: iterate-plan
description: Iterate on existing implementation plans with thorough research and updates
model: opus
effort: high
---

# Iterate Implementation Plan

You are tasked with updating an existing implementation plan based on user feedback. Be skeptical and thorough — changes must be grounded in what the codebase actually does, not in what the feedback assumes.

## Mark the herdr phase

As your very first action, tag this agent's herdr tab so the session navigator shows the workflow phase (safe no-op outside herdr):

```bash
bash "$(git rev-parse --show-toplevel)/.claude/scripts/herdr-phase.sh" plan
```

## Initial Response

Parse the input for a plan file path and the requested changes, then:

**Both provided** → proceed straight to Step 1.

**Plan file but no feedback:**

```
I've found the plan at [path]. What changes would you like to make?

For example:
- "Add a phase for migration handling"
- "Update the success criteria to include performance tests"
- "Split Phase 2 into two separate phases"
```

**Neither:**

```
I'll help you iterate on an existing implementation plan.

Which plan would you like to update? Please provide the path to the plan file
(e.g., `.rpi/2025-10-16-feature-plan.md`).

Tip: You can list recent plans with `ls -lt .rpi/*-plan.md | head`
```

## Process

### Step 1: Read and understand the current plan

Read the plan file COMPLETELY — no limit/offset. Understand its structure, phases, scope, and success criteria before touching anything. Then parse what the user wants added, modified, or removed, and judge whether it needs codebase research or is a pure editorial change.

### Step 2: Research, only if the change requires it

Simple changes (rewording criteria, splitting a phase, adjusting scope) need no research. Spawn agents only when the feedback requires new technical understanding or validation of an assumption.

When you do: use **codebase-locator** to find relevant files, **codebase-analyzer** for implementation details, **codebase-pattern-finder** for similar patterns, and the **artifact-*** agents for prior decisions. Run them in parallel, then read the files they surface fully into the main context before proceeding. Track the research with TodoWrite if it's more than a couple of tasks.

### Step 3: Present your understanding and confirm

```
Based on your feedback, I understand you want to:
- [Change 1 with specific detail]
- [Change 2 with specific detail]

My research found:
- [Relevant code pattern or constraint]
- [Important discovery that affects the change]

I plan to update the plan by:
1. [Specific modification to make]
2. [Another modification]

Does this align with your intent?
```

Get confirmation before editing.

### Step 4: Update the plan

Make surgical edits — preserve what doesn't need changing. Keep file:line references accurate, maintain the existing structure unless the change is structural, and keep new content at the same quality bar as the original: specific paths, measurable criteria, project-appropriate verification commands.

Check consistency as you go. A new phase should follow the existing phase pattern (including an empty `### Completion` block); a scope change should update "What We're NOT Doing"; an approach change should update "Implementation Approach."

**Never edit a filled-in `### Completion` block.** It's the record of what a finished phase actually did, often written by an agent that has since exited, and later phases read it as their only memory of earlier ones. Restructuring a phase that's already complete is the case to watch: carry its Completion block across intact. If your change invalidates something a completed phase recorded, say so in the *new* phase's spec rather than rewriting history.

### Step 5: Present the changes

```
I've updated the plan at `.rpi/[filename]`

Changes made:
- [Specific change 1]
- [Specific change 2]

The updated plan now:
- [Key improvement]

Would you like any further adjustments?
```

## Guidelines

**Be skeptical.** Question vague feedback and ask for clarification. If a requested change conflicts with an existing phase or seems technically infeasible, say so before making it — verifying with code research beats implementing a bad instruction cleanly.

**Be surgical.** Precise edits, not wholesale rewrites. Research only what the specific change requires.

**Be efficient with turns.** Confirm understanding before editing and allow course corrections, but batch clarifying questions rather than drip-feeding them — `AskUserQuestion` takes up to 4 multiple-choice questions per call, and free-text questions can go in one consolidated prompt.

**Leave no open questions.** If a change raises a question, resolve it before updating. The plan must stay fully actionable — an implementing agent will follow it without the context of this conversation.

## Success Criteria Structure

Preserve the two-category split when updating criteria:

**Automated** — commands an execution agent can run (the project's test, lint, and type-check commands), files that should exist, compilation success.

**Manual** — UI/UX behavior, performance under real conditions, edge cases that resist automation, user acceptance.
