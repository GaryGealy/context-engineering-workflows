# Template Adaptation

Reference for Step 5 of the setup skill.

Adapt each template by reasoning about what the project actually needs — read the template fully, find every project-specific reference, and rewrite those sections. Preserve the workflow logic and agent behaviors; change the tooling around them.

## What varies by project

**Commands.** Every `npm run test:unit`, `npm run lint`, `npm run format`, `npm run build`, and database command in the templates is a placeholder for whatever this project actually uses. Replace them with the detected commands, including in checklists and verification sections.

**Directory structure.** Adjust paths to the project's layout — `src/` vs `pkg/`/`internal/`, `tests/` vs `__tests__/`, framework-specific trees like `src/routes/`.

**Framework guidance.** Where a template gives framework-specific advice, replace it with advice for the detected framework — SvelteKit load functions and form actions, Django models/views/urls, Next.js app router and server actions, Axum extractors and response types. Keep it generic if no framework is detected.

**Database workflow.** Include the detected tool's develop-then-migrate cycle (Prisma `db push` → `migrate dev`, Django `makemigrations` → `migrate`, Alembic revisions, Diesel migrations). Remove database sections entirely if there's no database.

**Thoughts directory.** If disabled, strip thoughts-specific sections and skip the thoughts agents.

**Issue tracking.** Wire in the detected tracker's commands (`gh issue view`, `glab issue view`, Linear MCP/CLI, or local ticket file paths). If there's no tracker, remove ticket-specific references and fall back to generic "task description" language.

## branch-ticket-detector

The agent's "Issue Tracker" section has one subsection per tracker. **Keep only the subsection matching the detected tracker and delete the rest**, so its identifier pattern and fetch command match reality. For local files, set the project's actual ticket path.

**If issue tracking is None**, don't generate this agent at all. Then:
- In `research-codebase/SKILL.md`, strip the detection branch from "Initial Setup" and keep only the plain "ask the user for a research question" fallback
- Drop the "run it with no arguments" branch-detection mention from `guide/topics.md`

Otherwise the generated skills reference an agent that doesn't exist.

## The herdr-phase script

`reference/scripts/herdr-phase.sh` is project-agnostic — no commands, paths, or tooling to adapt. Copy it **verbatim** to `.claude/scripts/herdr-phase.sh` and `chmod +x` it. Do not rewrite it. It no-ops outside herdr, so install it unconditionally; each phase skill already calls it.

## What must survive adaptation

Four behaviors carry the workflow. Change the tooling around them freely; if any of them doesn't make it into the generated files, the adaptation failed.

**The documentarian split.** Research agents describe what exists; they don't critique or recommend. Design and planning are where opinions belong. Each agent states this once — in its `description` and one line of body — so it's easy to drop by accident while rewriting. Carry it through.

**Parallel sub-agent execution.** `/research-codebase` fans out across research questions in a single response. Preserve the fan-out instruction, not just the agent list — sequential research is the failure mode it exists to prevent.

**Interactive planning.** `/design` and `/create-plan` stop and confirm with the user at defined points: the design's open questions, the plan's phase outline. Keep those checkpoints, and keep the batching guidance that stops them turning into a question-per-turn drip.

**The automated vs. manual verification split.** Plans separate what an agent can verify itself from what needs a human. This distinction drives the pause-between-phases behavior in `/implement-plan`. Adapt the commands; keep the two categories.

## Judgment calls

- Multiple test commands → use the most comprehensive as the default, and keep the specific ones where the template distinguishes unit/integration/e2e
- No linter or formatter → omit those sections rather than leaving broken commands
- A Makefile with standard targets often beats the package manager's scripts — prefer whatever the team actually types
- Genuinely unclear → ask, don't guess

## The bar

Adapt by understanding, not by substitution. The templates describe a workflow; your job is to express that same workflow in this project's vocabulary. A generated file that mentions a command the project doesn't have, or a framework it doesn't use, is a failure even if every other line is correct.

The result should read as though it were written for this project, not like a generic template forced to fit.
