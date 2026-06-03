# Guide Topics

Present the content for the requested topic. Keep it practical and scannable.

## Topic: overview

### The Workflow

```
/research-codebase → /design → /create-plan → /implement-plan → /review-changes
                                /guide (run anytime)
```

This workflow uses **intentional compaction** — periodically pausing work and distilling progress into structured artifacts (research docs, designs, plans) before starting fresh context windows.

**Why it matters:** Your context window is the ONLY lever you have to affect output quality without retraining models.

**The five phases:**

1. **Research** (`/research-codebase`) — Explore the codebase. Sub-agents do the messy file discovery. Output: clean research document. Run it with no arguments and it detects the ticket from your current branch and researches against that.
2. **Design** (`/design`) — ~200-line alignment artifact. Current state, desired end state, patterns, testing approach. Your highest-leverage review moment.
3. **Plan** (`/create-plan`) — Vertical implementation phases with per-phase testing. Takes the design as input — decisions are already made.
4. **Implement** (`/implement-plan`) — Testing-aware, phase-by-phase execution. Generates review metadata as it goes.
5. **Review** (`/review-changes`) — Guided tour of the diff. Critical vs mechanical changes. Can post to PR.

**Strategic human review points:**

| Phase | Your Role | Impact |
|-------|-----------|--------|
| Design | Correct agent's thinking early | Prevents hundreds of lines of wrong code |
| Research | Validate findings are accurate | Prevents cascading errors |
| Planning | Review phasing and testing | Bad plan → bad implementation |
| Implementation | Manual testing between phases | Catch issues before they compound |
| Review | Focus on critical sections | Fast, targeted PR reviews |

**Real-world results:**
- 300k LOC Rust codebase: 1-hour bug fix by non-expert, PR approved without revision
- 35k LOC feature: 7 hours vs 3-5 days estimated, minimal PR revisions

## Topic: research

### Research Phase Deep Dive

**Purpose:** Thoroughly explore the codebase before making any design or implementation decisions.

**When to research:**
- Before starting any new feature
- Before fixing complex bugs
- Before refactoring
- When you don't understand how something works
- Even for "simple" tasks (prevents assumptions)

**What good research looks like:**
- Specific file paths with line numbers
- Explanation of data flow
- Identification of existing patterns (including test patterns)
- Examples of similar implementations
- Edge cases and gotchas discovered

**What bad research looks like:**
- Vague descriptions without file references
- Assumptions instead of verified facts
- Missing edge cases
- No examples of existing patterns
- Surface-level understanding

**Best practices:**
- Be specific in your research question
- Review the research document before designing
- Ask follow-up questions if unclear
- Validate findings match your understanding
- Look for multiple examples of patterns

## Topic: design

### Design Phase Deep Dive

**Purpose:** Create a lightweight alignment artifact before the full plan. Corrections here prevent hundreds of lines of wrong code.

**When to design:**
- After research, before planning
- For any non-trivial feature or change
- When you want to validate your understanding with the agent

**What a good design looks like:**
- ~200 lines (not 1000)
- Current state and desired end state clearly stated
- Patterns to follow explicitly called out
- Testing approach decided (not deferred to planning)
- Key decisions documented with rationale
- Scope boundaries defined (what we're NOT doing)
- All open questions resolved

**What a bad design looks like:**
- Just a restated ticket
- Includes implementation details or code snippets
- No testing approach
- Unresolved questions left in the doc
- Patterns not specified (agent will pick wrong ones)

**Best practices:**
- Read the research doc before starting
- Pay attention to "Patterns to Follow" — this is where you correct the agent
- The testing approach should match your project's actual infrastructure
- Use this as a shareable artifact — send to teammates for quick alignment

## Topic: plan

### Planning Phase Deep Dive

**Purpose:** Create a detailed spec that guides implementation. Design decisions are already made — this focuses on execution order and verification.

**What makes a good plan:**
- Specific file paths and line numbers
- Code examples showing the pattern
- Vertical phases (each delivers working, testable functionality)
- Per-phase testing aligned with the design's testing approach
- No open questions or "TBD" items

**What makes a bad plan:**
- Vague instructions like "implement feature X"
- Horizontal layers (all DB, then all API, then all UI)
- Testing deferred to a bottom section instead of per-phase
- Unresolved questions

**Planning workflow:**
1. Read the design document
2. Break into vertical phases (each delivers working functionality)
3. Add per-phase testing based on the design's testing approach
4. Detail each phase with specifics
5. Define success criteria per phase
6. Get approval — spot-check, not deep-review

## Topic: implement

### Implementation Phase Deep Dive

**Purpose:** Execute the plan with testing-aware implementation and verification at each step.

**Workflow:**
1. Read entire plan first
2. Implement Phase 1 completely (vertical slice)
3. Run per-phase tests as specified
4. Run automated verification
5. Pause for manual testing
6. Get confirmation, mark phase complete
7. Proceed to Phase 2

**Testing-aware implementation:**
- Follow the testing approach from the design doc
- Write tests as part of each phase, not after all phases
- Per-phase testing ensures each slice works before moving on
- Use the project's actual test infrastructure

**When to pause:**
- After each phase completes
- When context > 70% utilized
- When encountering unexpected complexity
- When tests are failing and unclear why
- When plan needs significant changes

**Resuming across sessions:**
1. Update plan with current status (checkboxes)
2. Start fresh context with the plan
3. Implementation picks up from last completed phase

## Topic: review

### Review Phase Deep Dive

**Purpose:** Make reviewing large PRs fast and focused. Get a guided tour of what matters.

**When to review:**
- After implementation is complete
- Before creating a PR
- When reviewing a coworker's PR

**The review guide tells you:**
- What's critical (read carefully)
- What's mechanical (safe to skim)
- What's tested and what's not
- Suggested review order
- Patterns to spot-check

**Best practices:**
- Run `/review-changes` before creating the PR
- Post the review guide as a PR comment for teammates
- Focus reading time on the "Critical Review" section
- Use the test coverage map to identify risk areas
- If something critical is untested, add tests before merging

## Topic: context

### Context Window Management

**Why context matters:** Your context window is the ONLY lever you have to affect AI output quality without retraining models.

**Optimization hierarchy:**
1. **Incorrect information** (most damaging) — wrong file paths, outdated code, false assumptions
2. **Missing information** — incomplete understanding, missing edge cases
3. **Excessive noise** — file search results, debug logs, tool outputs

**Target utilization: 40-60%**
- Greenfield features: 40-50% (need room for exploration)
- Bug fixes: 50-60% (more focused)
- Complex refactoring: 40% (lots of discovery)

**When to start fresh context:**
- Moving between phases (research → design → plan → implement)
- Completing a major implementation phase
- Context utilization > 70%
- Conversation became noisy with debugging

**What to carry forward:**
- Load the research/design/plan documents
- Reference specific findings
- Don't copy entire conversation history

**What gets compacted:**
- File search results → Research document
- Design discussion → Design document
- Implementation progress → Plan document (checkboxes)
- Debugging session → Updated plan

## Topic: patterns

### Common Workflow Patterns

**Greenfield Feature:**
```bash
/research-codebase "How are similar features implemented?"
/design thoughts/shared/research/2026-01-05-feature.md
/create-plan thoughts/shared/designs/2026-01-05-feature.md
/implement-plan thoughts/shared/plans/2026-01-05-feature.md
/review-changes
```

**Bug Fix:**
```bash
/research-codebase "Why is X failing?"
/design thoughts/shared/research/2026-01-05-bug.md
/create-plan thoughts/shared/designs/2026-01-05-bug.md
/implement-plan thoughts/shared/plans/2026-01-05-bug.md
/review-changes
```

**Refactoring:**
```bash
/research-codebase "How does module X work currently?"
/design thoughts/shared/research/2026-01-05-refactor.md
/create-plan thoughts/shared/designs/2026-01-05-refactor.md
/implement-plan thoughts/shared/plans/2026-01-05-refactor.md
/review-changes
```

**Multi-Day Feature:**
```bash
# Day 1: Research, design, plan
/research-codebase "How should feature X integrate?"
/design thoughts/shared/research/2026-01-05-feature.md
/create-plan thoughts/shared/designs/2026-01-05-feature.md

# Day 2+: Implement (resumes from last checkpoint)
/implement-plan thoughts/shared/plans/2026-01-05-feature.md

# Final: Review
/review-changes
```

**Iterating on a Plan:**
```bash
/iterate-plan thoughts/shared/plans/2026-01-05-feature.md
# "Split Phase 2 into two phases"
# "Update success criteria based on testing"
```

## Topic: tips

### Best Practices by Phase

**Research:**
- Be specific in questions
- Validate findings before designing
- Look for multiple pattern examples
- Include file:line references
- Don't skip research for "simple" tasks

**Design:**
- Keep it to ~200 lines
- Decide testing approach here, not later
- Explicitly call out patterns to follow
- Resolve all open questions
- Define what's NOT in scope

**Planning:**
- Include specific file paths
- Create vertical phases (testable slices)
- Include per-phase testing
- Resolve all questions before finalizing
- Don't write horizontal layers

**Implementation:**
- Complete one vertical phase at a time
- Run per-phase tests between phases
- Update checkboxes in plan
- Don't skip manual testing
- If blocked, update the plan — don't diverge

**Review:**
- Run /review-changes before creating PR
- Focus on "Critical Review" sections
- Check the test coverage map
- Share review guide with teammates

**Context Management:**
- Keep utilization 40-60%
- Compact into documents
- Start fresh contexts between phases
- Carry forward key documents, not conversation history

**Running on Claude Opus 4.7:**
- Treat the agent as a delegated engineer — give it complete context upfront rather than steering turn-by-turn
- `/implement-plan` is the ideal auto-mode candidate (Shift+Tab to toggle) — the plan provides the upfront context it needs
- Default effort level is `xhigh`; bump to `max` only for genuinely hard problems (it can overthink)
- If you want more reasoning: add "think carefully and step-by-step" to your prompt
- If you want faster responses: add "prioritize responding quickly rather than thinking deeply"
- The model spawns fewer subagents by default — if you want parallel fan-out for research, say so explicitly

## Topic: examples

### Real-World Success Stories

**300k LOC Rust Codebase:**
- Task: Fix a bug in large Rust codebase
- Developer: Non-expert in the codebase
- Time: 1 hour total
- Result: PR approved without revision
- Key lesson: Brownfield codebases are approachable with proper research

**Complex Feature (35k LOC):**
- Task: Add cancellation support + WASM compilation
- Estimated time: 3-5 days per senior engineer
- Actual time: 7 hours (3 research/planning, 4 implementation)
- Result: Both PRs completed with minimal revision
- Key lesson: Research time pays off exponentially

**Failure Case — Hadoop Dependencies:**
- Task: Remove dependencies from Parquet Java
- Issue: Insufficient dependency tree exploration
- Result: Failed to complete task
- Key lesson: Domain expertise matters; research depth requires adequate effort

## Attribution

This workflow is inspired by **HumanLayer's** research on AI-assisted development, with additional influences from **CRISPY/Dex** (design-before-planning, instruction budgets) and **Simon Willison** (TDD, conformance-driven development).

- [humanlayer.dev](https://humanlayer.dev)
- [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
- [AI Engineering Talk](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i)
