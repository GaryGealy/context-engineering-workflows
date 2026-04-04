---
name: workflow-guide
description: Learn how to use the Research → Design → Plan → Implement → Review workflow effectively through intentional compaction
---

# Research → Design → Plan → Implement → Review Workflow Guide

Interactive guide to understanding and using the RDPIR (Research, Design, Plan, Implement, Review) workflow effectively.

## How to Use This Guide

You can:
- Run `/workflow-guide` to see the full guide
- Run `/workflow-guide [topic]` to learn about a specific area

**Available topics:**
- `overview` - What is intentional compaction and the workflow?
- `research` - How to use the research phase effectively
- `design` - How to use the design phase effectively
- `plan` - How to create good implementation plans
- `implement` - How to execute plans successfully
- `review` - How to review changes efficiently
- `context` - Managing context windows and compaction
- `patterns` - Common workflow patterns (greenfield, bug fix, refactoring)
- `tips` - Best practices and common pitfalls
- `examples` - Real-world success stories

## Quick Start

If no topic specified, show this quick start guide:

---

# Research → Design → Plan → Implement → Review Workflow

This workflow uses **intentional compaction** to manage AI agent context windows effectively.

## What is Intentional Compaction?

Intentional compaction is a deliberate strategy where you periodically pause work and distill progress into structured artifacts (research summaries, designs, plans, status updates) before starting fresh context windows.

**Why it matters:** Since LLMs are stateless functions, your context window is the ONLY lever you have to affect output quality without retraining models.

## The Workflow

### 🔍 Phase 1: Research (`/research-codebase`)

**Purpose:** Explore and understand the codebase without polluting your main context.

**Example:**
```bash
/research-codebase "How does user authentication work?"
```

**What happens:**
- Spawns parallel sub-agents to explore
- Sub-agents search files, trace data flow, find patterns
- Output: Clean research document with findings
- Main agent never sees messy file discovery

**Output:** `thoughts/shared/research/YYYY-MM-DD-topic.md`

### 🎨 Phase 2: Design (`/design`)

**Purpose:** Create a ~200-line design discussion to align on what you're building. This is your highest-leverage review moment.

**Example:**
```bash
/design thoughts/shared/research/2026-01-05-auth-feature.md
```

**What happens:**
- Reads research document
- Proposes current state and desired end state
- Identifies patterns to follow
- Decides testing approach upfront
- Resolves open questions before planning

**Output:** `thoughts/shared/designs/YYYY-MM-DD-topic.md`

**Critical insight:** Corrections here prevent hundreds of lines of wrong code. This is cheaper than fixing a bad plan.

### 📋 Phase 3: Plan (`/create-plan`)

**Purpose:** Takes the design doc as input. Creates vertical implementation phases with per-phase testing.

**Example:**
```bash
/create-plan thoughts/shared/designs/2026-01-05-auth-feature.md
```

**What happens:**
- Reads design document (decisions already made)
- Creates vertical phases (each delivers a working slice)
- Includes per-phase testing based on the design's testing approach
- Defines success criteria for each phase

**Output:** `thoughts/shared/plans/YYYY-MM-DD-topic.md`

### ⚙️ Phase 4: Implement (`/implement-plan`)

**Purpose:** Testing-aware implementation. Follows the testing approach specified in the design and plan.

**Example:**
```bash
/implement-plan thoughts/shared/plans/2026-01-05-auth-feature.md
```

**What happens:**
- Reads plan completely
- Implements one vertical phase at a time
- Runs per-phase tests and automated verification
- Pauses for manual testing between phases
- Updates checkboxes in plan
- Generates review metadata for the review phase

### 🔎 Phase 5: Review (`/review-changes`)

**Purpose:** Generate a guided review of your changes. Highlights critical vs mechanical code, test coverage, and suggested review order.

**Example:**
```bash
/review-changes
```

**What happens:**
- Analyzes all changes made during implementation
- Categorizes code as critical or mechanical
- Maps test coverage
- Suggests optimal review order
- Generates a review guide you can share with teammates

### 🧭 Utility: Guide (`/guide`)

**Purpose:** Run anytime to see where you are in the workflow.

**Example:**
```bash
/guide
```

**What happens:**
- Shows your current position in the workflow
- Lists available artifacts (research docs, designs, plans)
- Suggests next steps

## Strategic Human Review Points

Focus your effort on the **highest-leverage checkpoints**:

| Phase | Your Role | Impact |
|-------|-----------|--------|
| Design | Correct agent's thinking early | Prevents hundreds of lines of wrong code |
| Research | Validate findings are accurate/complete | Prevents cascading errors |
| Planning | Review implementation approach | Bad plan → hundreds of bad lines |
| Implementation | Manual testing between phases | Catch issues before they compound |
| Review | Focus on critical sections | Fast, targeted PR reviews |

## Context Window Management

**Optimization Hierarchy** (prioritize by worst outcomes):
1. ❌ Incorrect information (most damaging)
2. ⚠️ Missing information
3. 📊 Excessive noise

**Target:** Keep context utilization at 40-60%
- Lower utilization preserves capacity for course corrections
- Avoid maxing out context—leaves no room for debugging

## Real-World Results

- **300k LOC Rust codebase:** 1-hour bug fix by non-expert, PR approved without revision
- **35k LOC feature:** 7 hours vs 3-5 days estimated, minimal PR revisions
- **Key factor:** Upfront research investment pays off

## Quick Tips

✓ Always research before designing (even for "simple" tasks)
✓ Use the design phase to align on approach — it's your cheapest correction point
✓ Review and validate plans before implementing
✓ Implement one phase at a time, verify between phases
✓ Run /review-changes before creating your PR
✓ Keep context utilization under 70%
✓ Compact progress into documents, start fresh contexts
✓ Use /guide anytime to check your current workflow position

## Learn More

Run `/workflow-guide [topic]` for detailed information:
- `/workflow-guide research` - Deep dive on research phase
- `/workflow-guide design` - Design phase best practices
- `/workflow-guide plan` - Planning best practices
- `/workflow-guide implement` - Implementation patterns
- `/workflow-guide review` - Review phase guide
- `/workflow-guide context` - Context window optimization
- `/workflow-guide patterns` - Common workflow patterns
- `/workflow-guide tips` - Best practices and pitfalls

---

## Topic-Specific Content

When user specifies a topic, provide detailed content for that area:

### Topic: `overview`

Show the quick start content above plus:

**What Gets Compacted:**
- File search results
- Code flow understanding
- Build/test logs
- Tool output (JSON blobs)
- Error states and debugging attempts
- Research findings
- Design discussions
- Implementation progress

**Compaction Output Format:**
```markdown
# [Topic] Research/Design/Plan/Status

## Problem Statement
[What we're trying to solve]

## Investigation Findings
[What we discovered]

## Current Status
[Where we are now]

## Next Steps
[What to do next]

## Critical Dependencies
[Important constraints or relationships]
```

### Topic: `research`

**Research Phase Deep Dive**

**Purpose:** Thoroughly explore the codebase before making any design or implementation decisions.

**When to research:**
- Before starting any new feature
- Before fixing complex bugs
- Before refactoring
- When you don't understand how something works
- Even for "simple" tasks (prevents assumptions)

**What good research looks like:**
✓ Specific file paths with line numbers
✓ Explanation of data flow
✓ Identification of existing patterns
✓ Examples of similar implementations
✓ Notes on conventions and standards
✓ Edge cases and gotchas discovered

**What bad research looks like:**
❌ Vague descriptions without file references
❌ Assumptions instead of verified facts
❌ Missing edge cases or error handling
❌ No examples of existing patterns
❌ Surface-level understanding

**Best practices:**
- Be specific in your research question
- Review the research document before designing
- Ask follow-up questions if unclear
- Validate findings match your understanding
- Look for multiple examples of patterns

**Example research questions:**
- "How does user authentication work in this codebase?"
- "Where are API endpoints defined and what patterns do they follow?"
- "How is error handling done in the payment processing module?"
- "What testing patterns exist for database migrations?"

### Topic: `design`

**Design Phase Deep Dive**

**Purpose:** Create a lightweight alignment artifact before the full plan. Corrections here prevent hundreds of lines of wrong code.

**When to design:**
- After research, before planning
- For any non-trivial feature or change
- When you want to validate your understanding with the agent

**What a good design looks like:**
✓ ~200 lines (not 1000)
✓ Current state and desired end state clearly stated
✓ Patterns to follow explicitly called out
✓ Testing approach decided (not deferred to planning)
✓ Key decisions documented with rationale
✓ Scope boundaries defined (what we're NOT doing)
✓ All open questions resolved

**What a bad design looks like:**
❌ Just a restated ticket
❌ Includes implementation details or code snippets
❌ No testing approach
❌ Unresolved questions left in the doc
❌ Patterns not specified (agent will pick wrong ones)

**Best practices:**
- Read the research doc before starting
- Pay attention to the "Patterns to Follow" section — this is where you correct the agent
- The testing approach should match your project's actual infrastructure
- Use this as a shareable artifact — send to teammates for quick alignment

### Topic: `plan`

**Planning Phase Deep Dive**

**Purpose:** Create a detailed, unambiguous specification that guides implementation. Design decisions are already made — this phase focuses on execution order and verification.

**What makes a good plan:**
✓ Specific file paths and line numbers
✓ Code examples showing the pattern
✓ Clear success criteria (automated + manual)
✓ Vertical phases (each delivers a working slice, 3-5 phases max)
✓ Per-phase testing aligned with the design's testing approach
✓ Database migrations clearly specified
✓ No open questions or "TBD" items

**What makes a bad plan:**
❌ Vague instructions like "implement feature X"
❌ No specific file references
❌ Unclear when "done"
❌ All-or-nothing (no phases)
❌ Testing deferred to a bottom section instead of per-phase
❌ Unresolved questions

**Planning workflow:**
1. Read the design document
2. Break into vertical phases (each delivers working functionality)
3. Add per-phase testing based on the design's testing approach
4. Detail each phase with specifics
5. Define success criteria per phase
6. Get final approval

**Success Criteria Format:**
```markdown
### Phase 1: [Name]

#### Implementation:
- [ ] File changes listed with specifics

#### Per-Phase Testing:
- [ ] Unit tests pass: npm run test:unit -- --filter=phase1
- [ ] Integration test: verify end-to-end slice works
- [ ] Manual verification: [specific check]

### Success Criteria (Overall)

#### Automated Verification:
- [ ] All tests pass: npm run test:unit
- [ ] No linting errors: npm run lint
- [ ] Type checking passes: npm run check
- [ ] Build succeeds: npm run build

#### Manual Verification:
- [ ] Feature works in UI as expected
- [ ] Performance acceptable with 1000+ items
- [ ] Error messages are user-friendly
- [ ] Works on mobile devices
```

**Critical:** Each vertical phase should be independently testable and deliver a working slice of functionality.

### Topic: `implement`

**Implementation Phase Deep Dive**

**Purpose:** Execute the plan systematically with testing-aware implementation and verification at each step.

**Implementation workflow:**
1. Read entire plan first (don't skip ahead)
2. Implement Phase 1 completely (vertical slice)
3. Run per-phase tests as specified in the plan
4. Run all automated verification
5. Pause for manual testing
6. Get user confirmation
7. Mark phase complete in plan
8. Proceed to Phase 2

**Testing-aware implementation:**
✓ Follow the testing approach specified in the design
✓ Write tests as part of each phase, not after all phases
✓ Per-phase testing ensures each vertical slice works before moving on
✓ Use the project's actual test infrastructure (don't invent new patterns)

**Review metadata generation:**
- As you implement, track which changes are critical vs mechanical
- Note what's tested and what's not
- This metadata feeds into the /review-changes phase

**Best practices:**
✓ Complete one vertical phase fully before moving to next
✓ Run per-phase tests after each phase
✓ Update checkboxes in the plan as you go
✓ Don't skip manual testing steps
✓ If blocked, update the plan—don't diverge
✓ For complex work, recompact into plan periodically

**Common pitfalls:**
❌ Implementing multiple phases before testing
❌ Skipping test failures to "come back later"
❌ Diverging from plan without updating it
❌ Not marking progress in plan
❌ Maxing out context window
❌ Deferring all tests to the end instead of per-phase

**When to pause implementation:**
- After each phase completes
- When context > 70% utilized
- When encountering unexpected complexity
- When tests are failing and unclear why
- When plan needs significant changes

**Recompaction pattern:**
If implementation spans multiple days or contexts:
1. Update plan with current status
2. Note what's complete, what's in progress
3. Document any discoveries or blockers
4. Start fresh context with updated plan

### Topic: `review`

**Review Phase Deep Dive**

**Purpose:** Make reviewing large PRs fast and focused. Get a guided tour of what matters.

**When to review:**
- After implementation is complete
- Before creating a PR
- When reviewing a coworker's PR

**The review guide tells you:**
✓ What's critical (read carefully)
✓ What's mechanical (safe to skim)
✓ What's tested and what's not
✓ Suggested review order
✓ Patterns to spot-check

**Best practices:**
- Run /review-changes before creating the PR
- Post the review guide as a PR comment for teammates
- Focus your reading time on the "Critical Review" section
- Use the test coverage map to identify risk areas
- If something critical is untested, add tests before merging

### Topic: `context`

**Context Window Management Deep Dive**

**Why context matters:**
Your context window is the ONLY lever you have to affect AI output quality without retraining models.

**Optimization hierarchy:**
1. ❌ **Incorrect information** (most damaging)
   - Wrong file paths, outdated code, false assumptions
   - Prevents: Careful verification, research phase

2. ⚠️ **Missing information**
   - Incomplete understanding, missing edge cases
   - Prevents: Thorough research, asking questions

3. 📊 **Excessive noise**
   - File search results, debug logs, tool outputs
   - Prevents: Compaction, sub-agents

**Target utilization: 40-60%**
- Greenfield features: 40-50% (need room for exploration)
- Bug fixes: 50-60% (more focused)
- Complex refactoring: 40% (lots of discovery)

**When to start fresh context:**
✓ Moving from research → design
✓ Moving from design → planning
✓ Moving from planning → implementation
✓ Completing a major phase
✓ Context utilization > 70%
✓ Conversation became noisy with debugging

**What to carry forward:**
- Load the research document
- Load the design document
- Load the plan document
- Reference specific findings
- Don't copy entire conversation history

**What gets compacted:**
- File search results → Research document
- Design discussion → Design document
- Implementation progress → Plan document (checkboxes)
- Debugging session → Updated plan or new research
- Error states → Status update in plan

### Topic: `patterns`

**Common Workflow Patterns**

### Pattern 1: Greenfield Feature

```bash
# 1. Research existing patterns
/research-codebase "How are similar features implemented?"

# 2. Design the approach
/design thoughts/shared/research/2026-01-05-feature-x.md

# 3. Create plan from design
/create-plan thoughts/shared/designs/2026-01-05-feature-x.md

# 4. Implement
/implement-plan thoughts/shared/plans/2026-01-05-feature-x.md

# 5. Review before PR
/review-changes
```

### Pattern 2: Bug Fix

```bash
# 1. Research to understand bug
/research-codebase "Why is X failing?"

# 2. Design the fix
/design thoughts/shared/research/2026-01-05-bug-123.md

# 3. Plan the fix
/create-plan thoughts/shared/designs/2026-01-05-bug-123.md

# 4. Implement with tests
/implement-plan thoughts/shared/plans/2026-01-05-fix-bug-123.md

# 5. Review changes
/review-changes
```

### Pattern 3: Refactoring

```bash
# 1. Research current implementation
/research-codebase "How does module X work currently?"

# 2. Design the refactoring approach
/design thoughts/shared/research/2026-01-05-refactor-x.md

# 3. Plan incremental changes
/create-plan thoughts/shared/designs/2026-01-05-refactor-x.md

# 4. Implement with backwards compatibility
/implement-plan thoughts/shared/plans/2026-01-05-refactor-x.md

# 5. Review changes
/review-changes
```

### Pattern 4: Complex Feature (Multi-Day)

```bash
# Day 1: Research, design, and planning
/research-codebase "How should feature X integrate?"
/design thoughts/shared/research/2026-01-05-feature-x.md
/create-plan thoughts/shared/designs/2026-01-05-feature-x.md

# Day 2: Implement Phase 1-2
/implement-plan thoughts/shared/plans/feature-x.md
# (Complete phases 1-2, update checkboxes)

# Day 3: Continue implementation
# Start fresh context, load plan
/implement-plan thoughts/shared/plans/feature-x.md
# (Continues from last completed phase)

# Day 3 (end): Review
/review-changes
```

### Pattern 5: Iterating on Plan

```bash
# After feedback or new discoveries
/iterate-plan thoughts/shared/plans/2026-01-05-feature-x.md

# Provide updates:
# - "Split Phase 2 into two phases"
# - "Add error handling for edge case Y"
# - "Update success criteria based on testing"
```

### Pattern 6: Quick Check

```bash
# Not sure where you are? Run guide anytime
/guide
```

### Topic: `tips`

**Best Practices and Common Pitfalls**

### Research Phase

**✓ Do:**
- Be specific in questions
- Validate findings before designing
- Look for multiple pattern examples
- Document edge cases
- Include file:line references

**✗ Don't:**
- Skip research for "simple" tasks
- Accept vague or unclear findings
- Rely on assumptions
- Stop at surface-level understanding

### Design Phase

**✓ Do:**
- Keep it to ~200 lines
- Decide the testing approach here, not later
- Explicitly call out patterns to follow
- Resolve all open questions
- Define what's NOT in scope

**✗ Don't:**
- Include implementation details or code snippets
- Leave questions unresolved
- Skip the testing approach
- Restate the ticket without adding value
- Let it grow to 1000+ lines

### Planning Phase

**✓ Do:**
- Include specific file paths
- Write measurable success criteria
- Create vertical phases (each delivers working functionality)
- Include per-phase testing
- Resolve all open questions before finalizing

**✗ Don't:**
- Write vague plans
- Leave questions unresolved
- Create all-or-nothing plans
- Defer all testing to a bottom section
- Forget database migrations

### Implementation Phase

**✓ Do:**
- Read entire plan first
- Complete one vertical phase at a time
- Run per-phase tests between phases
- Update checkboxes in plan
- Pause for manual testing

**✗ Don't:**
- Implement all phases before testing
- Skip test failures
- Diverge from plan without updating it
- Max out context window
- Rush manual testing

### Review Phase

**✓ Do:**
- Run /review-changes before creating PR
- Focus on the "Critical Review" sections
- Check the test coverage map
- Share the review guide with teammates

**✗ Don't:**
- Skip the review phase for "small" changes
- Treat all code as equally important to review
- Ignore untested critical sections

### Context Management

**✓ Do:**
- Keep utilization 40-60%
- Compact into documents
- Start fresh contexts regularly
- Carry forward key documents

**✗ Don't:**
- Let context max out
- Copy entire conversation history
- Keep all debugging in context
- Ignore context warnings

### Topic: `examples`

**Real-World Success Stories**

### Example 1: BAML 300k LOC Rust Codebase

**Task:** Fix a bug in large Rust codebase
**Developer:** Non-expert in the codebase
**Time:** 1 hour total
**Result:** PR approved without revision

**What worked:**
- Thorough research phase identified exact issue location
- Plan was simple and focused
- Implementation followed plan exactly
- Tests passed first try

**Key lesson:** Brownfield codebases are approachable with proper research

### Example 2: Complex Feature (35k LOC)

**Task:** Add cancellation support + WASM compilation
**Estimated time:** 3-5 days per senior engineer
**Actual time:** 7 hours (3 research/planning, 4 implementation)
**Result:** Both PRs completed with minimal revision

**What worked:**
- Upfront research investment (3 hours)
- Detailed plan with clear phases
- Incremental implementation
- Verification at each phase

**Key lesson:** Research time pays off exponentially

### Example 3: Failure Case - Hadoop Dependencies

**Task:** Remove dependencies from Parquet Java
**Issue:** Insufficient dependency tree exploration
**Result:** Failed to complete task

**What went wrong:**
- Research phase too shallow
- Didn't fully map dependency relationships
- Underestimated complexity
- Lacked domain expertise

**Key lesson:** Domain expertise matters; research depth requires adequate effort

## Measuring Success

**Good indicators:**
✓ Research documents consulted during design
✓ Designs are ~200 lines with clear decisions
✓ Plans have specific file:line references and vertical phases
✓ Implementation rarely diverges from plan
✓ Per-phase tests pass between phases
✓ Review guide highlights risk areas before PR
✓ PRs require minimal revision
✓ Context windows stay under 70%

**Warning signs:**
⚠️ Skipping research or design phases
⚠️ Vague plans without specifics
⚠️ Implementing without testing
⚠️ Context window maxing out
⚠️ Frequent plan divergence
⚠️ PRs need major revisions

## Summary

The Research → Design → Plan → Implement → Review workflow succeeds through:
1. **Thorough research** before making decisions
2. **Lightweight design** to align on approach and testing strategy
3. **Clear planning** with vertical phases and per-phase testing
4. **Incremental implementation** with testing-aware execution
5. **Guided review** to focus human attention on what matters
6. **Active engagement** at human checkpoints (especially design)
7. **Context management** through intentional compaction

**Remember:** This is not magic—it requires your active participation at the highest-leverage points. The design phase is your cheapest correction point.

## Attribution

This workflow is inspired by and adapted from **HumanLayer's** research and implementation patterns for AI-assisted development.

**Original inspiration:**
- **Website:** [humanlayer.dev](https://humanlayer.dev)
- **GitHub:** [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
- **Talk:** [AI Engineering Talk](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i) - Deep dive on context engineering for coding agents

**Additional resources:**
- [Advanced Context Engineering for Coding Agents](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents) - Detailed guide on the principles behind this workflow

The intentional compaction strategy and research → plan → implement pattern originated from HumanLayer's work on optimizing AI agent effectiveness through context window management.
