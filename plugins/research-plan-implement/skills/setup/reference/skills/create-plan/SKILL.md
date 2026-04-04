---
name: create-plan
description: Create vertical implementation plans with per-phase testing from a design document
model: opus
effort: high
---

# Implementation Plan

You are tasked with creating detailed, vertical implementation plans from an approved design document. The design decisions are already made — your job is to translate them into actionable phases.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided:**
   - If a design doc path was provided as $ARGUMENTS, read it FULLY
   - Also check for and read any referenced research docs or tickets
   - Begin the planning process

2. **If no parameters provided**, respond with:

   ```
   I'll help you create an implementation plan. Please provide:
   1. The design document (from /design) — required
   2. Optionally, the research doc or ticket reference

   Tip: /create-plan thoughts/shared/designs/2026-01-05-auth-redesign.md
   ```

   Then wait for the user's input.

## Process

### Step 1: Context Gathering

1. **Read the design document FULLY** — this is your primary input
2. **Read any referenced research docs and tickets**
3. **Spawn focused research agents if needed:**
   - Use **codebase-locator** to find specific files referenced in the design
   - Use **codebase-analyzer** to understand implementation details needed for planning
   - Only research what the design doc doesn't already cover
4. **Read all files identified by research agents**

### Step 2: Create Plan Outline

Present a high-level outline of vertical phases before writing the full plan:

```
Based on the design document, here's my proposed phasing:

Phase 1: [Name] — [What it accomplishes end-to-end]
  Testing: [How this phase will be verified]

Phase 2: [Name] — [What it accomplishes end-to-end]
  Testing: [How this phase will be verified]

Phase 3: [Name] — [What it accomplishes end-to-end]
  Testing: [How this phase will be verified]

Does this phasing make sense? Should I adjust?
```

Get user approval on the outline before writing the full plan.

### Step 3: Write the Plan

Write the plan to `thoughts/shared/plans/YYYY-MM-DD-description.md`

**Filename format:** `YYYY-MM-DD-[ENG-XXXX-]description.md`
- YYYY-MM-DD is today's date
- ENG-XXXX is the ticket number (omit if no ticket)
- description is a brief kebab-case description

**Use the template from `template.md` in this skill's directory.** The template defines the plan structure with vertical phases, per-phase verification, and design doc references. Adapt it to the specific project and feature.

### Step 4: Present and Iterate

```
I've created the implementation plan at:
`thoughts/shared/plans/YYYY-MM-DD-description.md`

Please spot-check it — the design decisions are already aligned,
so this is about verifying the phasing and testing approach make sense.
```

Iterate based on feedback.

## Vertical Phase Design

**CRITICAL: Every phase must be a vertical slice, not a horizontal layer.**

Instead of:
```
Phase 1: All database changes
Phase 2: All service layer changes
Phase 3: All API changes
Phase 4: All frontend changes
Phase 5: All tests
```

Write:
```
Phase 1: Single feature working end-to-end (DB -> service -> API -> UI) with tests
Phase 2: Add complexity to that feature, with tests
Phase 3: Next feature end-to-end, with tests
```

**Why:** Each vertical phase is independently testable. If Phase 2 breaks, you know where the problem is. Horizontal phases mean nothing works until everything works.

## Per-Phase Testing

**Testing is not a section at the bottom. It's part of every phase.**

Each phase's Verification section should specify:
- What tests to write (drawn from the design doc's Testing Approach)
- What commands to run
- What "green" looks like — the concrete signal that this phase is working

Adapt the testing approach from the design doc:
- If the design says TDD: write failing tests first in the Changes Required section
- If the design says conformance testing: define the test suite as part of the phase
- If the design says manual testing: specify what to exercise and what to observe
- If the design doesn't specify: run whatever automated checks exist (lint, typecheck, build)

## Important Guidelines

1. **Design decisions are already made** — Don't re-litigate. The design doc has the rationale.
2. **Keep it tactical** — This is for the implementing agent. File paths, code snippets, commands.
3. **Vertical slices** — Every phase must be independently testable.
4. **Per-phase testing** — No "Testing Strategy" section at the bottom.
5. **Shorter than v1 plans** — Design alignment is already done. Focus on the how, not the what or why.
6. **Spot-check, don't deep-review** — Tell the user to spot-check, not spend an hour reviewing.
7. **No open questions** — If something is unclear, go back to the design doc or ask.
