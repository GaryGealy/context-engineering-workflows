---
name: implement-plan
description: Implement technical plans from thoughts/shared/plans with verification
model: opus
effort: xhigh
---

# Implement Plan

You are tasked with implementing an approved technical plan from `thoughts/shared/plans/`. These plans contain phases with specific changes and success criteria.

## Getting Started

When given a plan path:

- Read the plan completely and check for any existing checkmarks (- [x])
- Read the original ticket and all files mentioned in the plan
- **Read files fully** - never use limit/offset parameters, you need complete context
- Think carefully and step-by-step about how the pieces fit together — the plan is the spec, but real code rarely matches the plan exactly
- Create a todo list to track your progress
- Start implementing if you understand what needs to be done

If no plan path provided, ask for one.

**This skill is designed for long-running autonomous execution.** The plan provides complete upfront context — phases, file paths, verification commands — so the implementing agent can work end-to-end without progressive guidance. Users running this skill should consider enabling auto mode (Shift+Tab) to cut cycle time on multi-phase implementations.

## Implementation Philosophy

Plans are carefully designed, but reality can be messy. Your job is to:

- Follow the plan's intent while adapting to what you find
- Implement each phase fully before moving to the next
- Verify your work makes sense in the broader codebase context
- Update checkboxes in the plan as you complete sections

## Testing-Aware Implementation

Before implementing each phase, read its Verification section and adapt your workflow:

**If the phase specifies TDD / red-green testing:**
1. Write the failing tests first
2. Run them to confirm they fail
3. Implement the minimum code to make them pass
4. Run tests again to confirm green
5. Refactor if needed, keeping tests green
6. Commit tests and implementation separately when practical — this makes the red-to-green progression visible in the diff

**If the phase specifies conformance testing:**
1. Generate the test suite that defines correct behavior
2. Run it to confirm it fails (the feature doesn't exist yet)
3. Implement against the test suite
4. Run tests until green

**If the phase specifies manual testing:**
1. Implement the phase
2. Exercise the work yourself — start a server, run curl commands, call the API
3. Report what you observed in your phase completion message
4. Include the commands you ran and their output

**If the phase has no specific testing approach:**
1. Implement the phase
2. Run whatever automated checks exist (lint, typecheck, build, existing tests)

The testing approach was decided in `/design` and specified per-phase in `/create-plan`. Follow it — don't decide on a different approach.

## Review Metadata

As you implement each phase, keep a lightweight log for `/review-changes`. After completing all phases (or when the user runs `/review-changes`), save this to `thoughts/shared/review-metadata/YYYY-MM-DD-description.md` using the template from `review-metadata-template.md` in this skill's directory.

This metadata makes `/review-changes` much more accurate, but it's optional — `/review-changes` works without it by analyzing the diff directly.

## Verification Approach

After implementing a phase:

- Run the success criteria checks (usually `make check test` covers everything)
- Fix any issues before proceeding
- Update your progress in both the plan and your todos
- Check off completed items in the plan file itself using Edit
- **Pause for human verification**: After completing all automated verification for a phase, pause and inform the human that the phase is ready for manual testing. Use this format:

  ```
  Phase [N] Complete - Ready for Manual Verification

  Automated verification passed:
  - [List automated checks that passed]

  Please perform the manual verification steps listed in the plan:
  - [List manual verification items from the plan]

  Let me know when manual testing is complete so I can proceed to Phase [N+1].
  ```

If instructed to execute multiple phases consecutively, skip the pause until the last phase. Otherwise, assume you are just doing one phase.

do not check off items in the manual testing steps until confirmed by the user.

## When Things Don't Match the Plan

When things don't match the plan exactly:

- **Small deviations** (function signature changed, slightly different file path): Note it in your review metadata and keep going. Mention it in your phase completion message.
- **Structural deviations** (approach won't work, missing dependency, wrong assumption): STOP and present the issue clearly:

  ```
  Issue in Phase [N]:
  Expected: [what the plan says]
  Found: [actual situation]
  Why this matters: [explanation]

  Options:
  1. Adapt and continue (if the change is contained)
  2. Run /iterate-plan to update the plan
  ```

The plan is your guide, but your judgment matters. Small adaptations are fine. Structural changes need alignment.

## If You Get Stuck

When something isn't working as expected:

- First, make sure you've read and understood all the relevant code
- Consider if the codebase has evolved since the plan was written
- Present the mismatch clearly and ask for guidance

Use sub-tasks sparingly - mainly for targeted debugging or exploring unfamiliar territory.

## Resuming Work

If the plan has existing checkmarks:

- Trust that completed work is done
- Pick up from the first unchecked item
- Verify previous work only if something seems off

Remember: You're implementing a solution, not just checking boxes. Keep the end goal in mind and maintain forward momentum.
