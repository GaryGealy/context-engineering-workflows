---
name: setup
description: Generate project-specific research/plan/implement workflow by analyzing your project and intelligently adapting reference templates
model: opus
effort: high
---

# Setup Research-Plan-Implement Workflow

This skill analyzes your project's language, framework, and tooling, then intelligently adapts reference templates to create customized commands and agents in your `.claude/` directory.

## What This Skill Does

1. **Analyzes your project** by reading configuration files
2. **Adapts reference templates** intelligently based on what it finds
3. **Generates customized workflow** in `.claude/` directory
4. **No templates or hardcoded logic** - uses AI reasoning to adapt

## Before You Start

Before running setup, recommend the user create a branch for their work:

```
Tip: I recommend creating a branch before we set up the workflow — that way
you can review the generated files as a diff before merging them into your project.

  git checkout -b setup-rpi-workflow
```

If they're already on a feature branch, that's fine — just make sure they're aware that setup will create files in `.claude/`.

## Workflow

### Step 1: Analyze Current Project

Read project configuration files to understand the stack:

**Files to check (read what exists):**
- `package.json` - For Node/TypeScript projects
- `Cargo.toml` - For Rust projects
- `go.mod` - For Go projects
- `pyproject.toml` or `requirements.txt` - For Python projects
- `Makefile` - For build commands
- Check if `thoughts/` directory exists

**Extract key information:**
- Primary language and framework
- Package manager and available scripts
- Test commands (unit, integration, e2e)
- Linting and formatting commands
- Build and type-checking commands
- Database tools (Prisma, SQLAlchemy, Diesel, etc.)
- Directory structure conventions
- Issue/ticket tracking system:
  - Check for Linear CLI installation: `which linear`
  - Check for GitHub CLI: `which gh`
  - Check for GitLab CLI: `which glab`
  - Check for thoughts/tickets/ or thoughts/*/tickets/ directory
  - Check .git/config for repo info (github.com, gitlab.com, linear.app)

**Present findings to user:**
```
Detected project configuration:
- Language: [TypeScript/Python/Go/Rust]
- Framework: [SvelteKit/Next.js/Django/FastAPI/etc. or None]
- Package Manager: [npm/cargo/go/pip/poetry]
- Test Command: [npm run test:unit / cargo test / pytest / go test]
- Lint Command: [npm run lint / cargo clippy / ruff / golangci-lint]
- Format Command: [npm run format / cargo fmt / black / gofmt]
- Build Command: [npm run build / cargo build / make build]
- Database: [Prisma v6 / SQLAlchemy / Diesel / None]
- Issue Tracking: [Linear / GitHub Issues / GitLab Issues / Local Files / None]
- Has thoughts/ directory: [Yes/No]
```

**If anything couldn't be detected, mark it as:**
```
- Test Command: [Unable to detect - will ask]
- Type Check Command: [Unable to detect - will ask]
- Issue Tracking: [Unable to detect - will ask]
```

**Check for existing RPI installation:**
- Check if `.claude/skills/research-codebase/SKILL.md` exists (v2 installation)
- Check if `.claude/commands/research-codebase.md` exists (v1 installation)
- If v2 found: this is a **v2 upgrade** — enter upgrade mode (see Upgrade Mode section below)
- If v1 found: this is a **v1→v2 migration** — enter upgrade mode with migration (see Upgrade Mode section below)
- If neither found: this is a **fresh install** — continue with Step 2

### Step 2: Fill in Gaps with User Questions

For any commands or configuration that couldn't be auto-detected, ask the user specific questions. **Batch all gap-filling questions into a single AskUserQuestion call** rather than asking one-at-a-time — every user turn adds reasoning overhead, and the user would rather answer 6 questions once than 6 times. Only split into multiple rounds if a later question genuinely depends on an earlier answer (e.g., issue-tracker choice drives follow-up CLI questions).

The per-gap prompt text below shows the exact phrasing for each question — combine the ones that apply into one batched call.

**If test command not found:**
```
I couldn't find a test command. How do you run tests in this project?
- Example: npm test, cargo test, pytest, go test ./..., make test
- Or type 'none' if no tests yet
```

**If unit test command not found (but general test exists):**
```
I found a test command ([detected command]), but no specific unit test command.
How do you run unit tests specifically?
- Example: npm run test:unit, cargo test --lib, pytest tests/unit
- Or press Enter to use: [detected command]
```

**If integration test command not found:**
```
How do you run integration tests? (optional)
- Example: npm run test:integration, pytest tests/integration
- Or press Enter to skip
```

**If e2e test command not found:**
```
How do you run end-to-end tests? (optional)
- Example: npm run test:e2e, playwright test
- Or press Enter to skip
```

**If lint command not found:**
```
I couldn't find a linting command. How do you lint code?
- Example: npm run lint, cargo clippy, ruff check ., make lint
- Or type 'none' if not using a linter
```

**If format command not found:**
```
I couldn't find a formatting command. How do you format code?
- Example: npm run format, cargo fmt, black ., prettier --write .
- Or type 'none' if not using a formatter
```

**If type check command not found (for typed languages):**
```
How do you type-check your code?
- Example: npm run check, tsc --noEmit, mypy .
- Or press Enter to skip if not applicable
```

**If build command not found:**
```
How do you build your project?
- Example: npm run build, cargo build, make build, go build
- Or type 'none' if no build step needed
```

**If database detected but migration command unclear:**
```
I detected [database name] but couldn't determine migration commands.
How do you:
1. Apply schema changes during development?
   Example: npx prisma db push, python manage.py migrate
2. Create formal migrations?
   Example: npx prisma migrate dev, python manage.py makemigrations
```

**If issue tracking system not detected:**
```
How do you manage issues/tickets for this project?
1. Linear (linear.app)
2. GitHub Issues
3. GitLab Issues
4. Jira
5. Local files (e.g., thoughts/tickets/)
6. Other (please specify)
7. None - no formal issue tracking

Choice: [1-7]
```

**Follow-up based on choice:**

If Linear (1):
```
I'll generate Linear integration commands for:
- Reading ticket details
- Updating ticket status
- Creating research/plans linked to tickets

Linear CLI detected: [Yes/No]
If No: "Install with: npm install -g @linear/cli"
```

If GitHub Issues (2):
```
I'll generate GitHub Issues integration using gh CLI.
GitHub CLI detected: [Yes/No]
If No: "Install from: https://cli.github.com"
```

If GitLab Issues (3):
```
I'll generate GitLab Issues integration using glab CLI.
GitLab CLI detected: [Yes/No]
If No: "Install from: https://gitlab.com/gitlab-org/cli"
```

If Local files (5):
```
Where do you store ticket files?
- Default: thoughts/[username]/tickets/
- Custom path: [enter path]
```

If Other (6):
```
Please describe your issue tracking system:
(I'll generate generic workflow without specific integrations)
```

If None (7):
```
No issue tracking integration will be generated.
Plans and research can still reference work items manually.
```

**After filling gaps, present complete configuration:**
```
Complete project configuration:
✓ Language: TypeScript
✓ Framework: SvelteKit
✓ Package Manager: npm
✓ Test (unit): npm run test:unit
✓ Test (integration): npm run test:integration
✓ Test (e2e): playwright test
✓ Lint: npm run lint
✓ Format: npm run format
✓ Type Check: npm run check
✓ Build: npm run build
✓ Database: Prisma v6
  - Push schema: npx prisma@6 db push
  - Create migration: npx prisma@6 migrate dev
✓ Issue Tracking: GitHub Issues (gh CLI available)
✓ thoughts/ directory: Yes

Does this look correct?
```

### Step 3: Ask User Preferences

Ask about configuration choices:

1. **Thoughts Directory:**
   - If `thoughts/` exists: "I found a thoughts/ directory. Use it for research and plans? (yes/no)"
   - If not: "Would you like to use a thoughts/ directory for storing research and plans? (yes/no)"
   - If yes: "What structure? (shared/research/ and shared/plans/ or custom?)"

2. **Additional Commands:**
   - "Any additional custom verification commands I should include?"
   - "Any project-specific testing notes or requirements?"

3. **Thoughts gitignore:**
   - Check if `thoughts/` is in `.gitignore`
   - If not: "I recommend adding `thoughts/` to `.gitignore`. These are working artifacts, not source code — keeping them out of git keeps your PRs clean. Add it? (yes/no)"
   - If yes and thoughts files are already tracked in git:
     "I see thoughts files are currently tracked. Want me to untrack them? This removes them from git tracking without deleting the local files. (yes/no)"
     - If yes: run `git rm --cached -r thoughts/`

4. **Final Confirmation:**
   - "Ready to generate workflow files? (yes/no)"

### Step 4: Read Reference Templates

Read all reference templates from the plugin:

**Skills to read:**
- `skills/setup/reference/skills/research-codebase/SKILL.md`
- `skills/setup/reference/skills/design/SKILL.md`
- `skills/setup/reference/skills/create-plan/SKILL.md`
- `skills/setup/reference/skills/iterate-plan/SKILL.md`
- `skills/setup/reference/skills/implement-plan/SKILL.md`
- `skills/setup/reference/skills/review-changes/SKILL.md`
- `skills/setup/reference/skills/guide/SKILL.md`

**Agents to read:**
- `skills/setup/reference/agents/codebase-analyzer.md`
- `skills/setup/reference/agents/codebase-locator.md`
- `skills/setup/reference/agents/codebase-pattern-finder.md`
- `skills/setup/reference/agents/query-planner.md`
- `skills/setup/reference/agents/web-search-researcher.md`

**Conditional agents (if user wants thoughts/):**
- `skills/setup/reference/agents/thoughts-analyzer.md`
- `skills/setup/reference/agents/thoughts-locator.md`

### Step 5: Intelligently Adapt Each Template

For each template, reason about what needs to change based on the project analysis:

**Common adaptations needed:**

1. **Command Replacements:**
   - `npm run test:unit` → `cargo test --lib` (Rust)
   - `npm run test:unit` → `pytest tests/unit` (Python)
   - `npm run test:unit` → `go test ./...` (Go)
   - `npm run lint` → `cargo clippy` (Rust)
   - `npm run lint` → `ruff check .` (Python)
   - `npm run format` → `cargo fmt` (Rust)
   - `npm run format` → `black .` (Python)
   - `npx prisma@6 db push` → `python manage.py migrate` (Django)
   - `npx prisma@6 db push` → `diesel migration run` (Rust)

2. **Directory Structure:**
   - `src/` → `pkg/` or `internal/` (Go)
   - `tests/` → `__tests__/` (some Node projects)
   - Framework-specific paths (e.g., `src/routes/` for SvelteKit)

3. **Framework-Specific Guidance:**
   - SvelteKit: Add notes about load functions, form actions, remote functions
   - Django: Add notes about models, views, urls patterns
   - Next.js: Add notes about app router, server actions
   - Keep generic if no framework detected

4. **Database Workflow:**
   - Prisma: Include `npx prisma@6 db push` and `npx prisma@6 migrate dev` workflow
   - Django: Include `makemigrations` and `migrate` workflow
   - SQLAlchemy: Include Alembic migration workflow
   - Remove database sections if no database detected

5. **Thoughts Directory:**
   - If enabled: Keep all thoughts/ references
   - If disabled: Remove thoughts-specific sections, skip thoughts agents

6. **Issue Tracking Integration:**
   - If Linear: Add references to Linear ticket reading, status updates
   - If GitHub: Add references to `gh issue view`, `gh issue list`
   - If GitLab: Add references to `glab issue view`, `glab issue list`
   - If Local files: Reference ticket file paths (e.g., `thoughts/tickets/ENG-123.md`)
   - If None: Remove ticket-specific references, keep generic "task description"

**Adaptation process:**
- Read each template completely
- Identify all project-specific references (commands, paths, tools)
- Intelligently rewrite those sections for the detected project
- Preserve the core workflow logic and agent behaviors
- Maintain the documentarian philosophy (document what IS, not what SHOULD BE)

### Step 6: Write Adapted Files

Create the `.claude/` directory structure if it doesn't exist, then write adapted files:

**Directory structure to create:**
```
.claude/
├── skills/
│   ├── research-codebase/
│   │   └── SKILL.md
│   ├── design/
│   │   └── SKILL.md
│   ├── create-plan/
│   │   └── SKILL.md
│   ├── iterate-plan/
│   │   └── SKILL.md
│   ├── implement-plan/
│   │   └── SKILL.md
│   ├── review-changes/
│   │   └── SKILL.md
│   ├── guide/
│   │   └── SKILL.md
│   └── read-ticket/              # Optional: if Linear/GitHub/GitLab
│       └── SKILL.md
└── agents/
    ├── codebase-analyzer.md
    ├── codebase-locator.md
    ├── codebase-pattern-finder.md
    ├── query-planner.md
    ├── web-search-researcher.md
    ├── thoughts-analyzer.md      # Only if thoughts/ enabled
    ├── thoughts-locator.md       # Only if thoughts/ enabled
    └── ticket-reader.md          # Optional: if Linear/GitHub/GitLab
```

**Optional files based on issue tracking:**

If using **Linear**, optionally generate:
- `skills/read-ticket/SKILL.md` - Read Linear ticket details and save to thoughts/tickets/
- `agents/linear-ticket-reader.md` - Agent specialized in fetching Linear tickets
- Note: Only generate if user wants Linear integration

If using **GitHub Issues**, optionally generate:
- `skills/read-issue/SKILL.md` - Read GitHub issue using `gh issue view`
- Note: Uses gh CLI, no special agent needed

If using **GitLab Issues**, optionally generate:
- `skills/read-issue/SKILL.md` - Read GitLab issue using `glab issue view`
- Note: Uses glab CLI, no special agent needed

**Write each adapted skill and agent file:**
- Use Write tool to create each file
- Ensure adapted content matches the project's actual tooling
- Preserve markdown formatting and frontmatter

### Step 7: Present Summary and Next Steps

Show the user what was created:

```
Created research-design-plan-implement workflow in .claude/

Generated files:
- 7 skills: /research-codebase, /design, /create-plan, /iterate-plan, /implement-plan, /review-changes, /guide
- [N] agents: query-planner, codebase-locator, codebase-analyzer, pattern-finder, web-search-researcher, [+thoughts agents if enabled]

Adapted for your project:
- Test command: [detected command]
- Lint command: [detected command]
- Format command: [detected command]
- Build command: [detected command]
- Database: [detected tool and commands]
- Issue tracking: [detected system]

Workflow:
  /research-codebase -> /design -> /create-plan -> /implement-plan -> /review-changes
  /guide (run anytime for orientation)

Quick start:
  /research-codebase "How does authentication work?"
  /design thoughts/shared/research/2026-01-05-auth-flow.md
  /create-plan thoughts/shared/designs/2026-01-05-auth-redesign.md
  /implement-plan thoughts/shared/plans/2026-01-05-auth-redesign.md
  /review-changes
  /guide
```

### Step 8: Show Workflow Quick Tips

After presenting the summary, show essential workflow concepts:

```
Understanding the Research -> Design -> Plan -> Implement -> Review Workflow

This workflow uses "INTENTIONAL COMPACTION" to manage context windows:

RESEARCH Phase (/research-codebase)
   Explores codebase without polluting main context
   Sub-agents handle messy file discovery
   Output: Clean research document with findings

DESIGN Phase (/design)
   Lightweight ~200-line alignment artifact
   Captures: current state, desired end state, patterns, testing approach
   This is your highest-leverage review moment
   Output: Design discussion document

PLAN Phase (/create-plan)
   Takes design as input — decisions already made
   Vertical phases (end-to-end slices, not horizontal layers)
   Per-phase testing baked in
   Output: Tactical implementation plan

IMPLEMENT Phase (/implement-plan)
   Testing-aware: follows the testing approach from design/plan
   Generates review metadata as it goes
   Phase-by-phase with verification checkpoints

REVIEW Phase (/review-changes)
   Guides human attention through the diff
   Critical vs mechanical changes
   Test coverage map
   Can post review guide to PR

ORIENTATION (/guide)
   Run anytime to see where you are and what's next
   5-10 lines of contextual help

Key Success Factors:
  Always research before designing
  Design is your highest-leverage review moment
  Plans use vertical slices with per-phase testing
  Run /review-changes before or after creating a PR
  Run /guide if you forget where you are

Attribution:
  Inspired by HumanLayer's research on AI-assisted development
  Informed by talks from Dex (CRISPY) and Simon Willison (TDD/conformance)
  Website: humanlayer.dev
  GitHub: github.com/humanlayer/humanlayer
```

### Step 9: Optional - Create thoughts/ Directory

If user said yes to thoughts/ but it doesn't exist:

```
Would you like me to create the thoughts/ directory structure?
  thoughts/
  ├── shared/
  │   ├── research/
  │   ├── designs/
  │   ├── plans/
  │   └── review-metadata/
  └── [username]/
      ├── tickets/
      └── notes/
```

## Upgrade Mode

When existing RPI commands or skills are detected, enter upgrade mode:

### Step U1: Extract Project Adaptations

Read the existing commands/skills to extract project-specific details. Check both `.claude/commands/` (v1) and `.claude/skills/` (v2) locations:
- Test commands (unit, integration, e2e)
- Lint, format, build, typecheck commands
- Database tools and migration commands
- Framework-specific patterns (SvelteKit load functions, Django views, etc.)
- Issue tracking integration
- Thoughts directory configuration
- Any custom additions the user made

### Step U2: Show Upgrade Summary

```
Detected existing RPI installation. Here's what's changing:

MIGRATION:
  Commands are moving from .claude/commands/ to .claude/skills/ (new Claude Code convention)
  Old command files will be removed after migration

NEW skills:
  /design — Lightweight design discussion before planning (~200 lines vs ~1000 line plans)
  /review-changes — Structured review guide for PRs (critical vs mechanical changes)
  /guide — Contextual orientation (where am I? what's next?)

NEW agent:
  query-planner — Keeps research objective by separating questions from intent

UPDATED skills:
  /research-codebase — Now uses query planning to keep research objective
  /create-plan — Slimmed down (design decisions moved to /design), vertical phases, per-phase testing
  /implement-plan — Testing-aware (adapts to TDD/conformance/manual), generates review metadata

UNCHANGED:
  /iterate-plan — Migrated to skills format, content unchanged

Your project adaptations will be preserved:
  - Test command: [extracted]
  - Lint command: [extracted]
  - [etc.]

Ready to upgrade? (yes / let me see details for a specific skill)
```

### Step U3: Regenerate Files

1. Read all new reference templates (Step 4)
2. Adapt each template using the extracted project-specific details (Step 5)
3. Write all files to `.claude/skills/` and `.claude/agents/` (Step 6)
4. Remove old `.claude/commands/` files that have been migrated to skills:
   - `rm .claude/commands/research-codebase.md` (if exists)
   - `rm .claude/commands/create-plan.md` (if exists)
   - `rm .claude/commands/iterate-plan.md` (if exists)
   - `rm .claude/commands/implement-plan.md` (if exists)
   - `rm .claude/commands/read-ticket.md` (if exists)
   - Ask before removing: "I'll clean up the old .claude/commands/ files that have been migrated to .claude/skills/. OK? (yes/no)"
5. Handle thoughts gitignore (if not already configured)
6. Show updated summary (Step 7) and workflow tips (Step 8)

### Step U4: Create designs directory

If `thoughts/shared/` exists but `thoughts/shared/designs/` doesn't:
```
mkdir -p thoughts/shared/designs thoughts/shared/review-metadata
```

## Important Guidelines

**Be Intelligent, Not Mechanical:**
- Don't use string replacement or templates
- Use your understanding of the codebase to adapt
- Reason about what makes sense for this project
- Preserve the intent and workflow, not just the syntax

**Preserve Core Patterns:**
- Keep the documentarian philosophy (document what IS)
- Keep the parallel sub-agent execution pattern
- Keep the interactive planning approach
- Keep the automated vs manual verification distinction

**Handle Edge Cases:**
- If multiple test commands exist, use the most comprehensive
- If no linter/formatter found, omit those sections
- If unclear about something, ask the user
- If project has a Makefile, check if it has standard targets (test, lint, build)

**Don't Overwrite Existing Files:**
- Check if `.claude/skills/` or `.claude/commands/` directory already exists
- If skills exist, enter upgrade mode (see Upgrade Mode section)
- If only commands exist, enter migration + upgrade mode
- If selective, show list and let user choose which to regenerate

## Example Adaptations

### TypeScript/SvelteKit → Python/Django

**Before (from reference):**
```markdown
- [ ] Schema changes apply cleanly: `npx prisma@6 db push`
- [ ] All unit tests pass: `npm run test:unit`
- [ ] No linting errors: `npm run lint`
```

**After (adapted):**
```markdown
- [ ] Migrations apply cleanly: `python manage.py migrate`
- [ ] All unit tests pass: `pytest tests/unit`
- [ ] No linting errors: `ruff check .`
```

### TypeScript/SvelteKit → Rust/Axum

**Before:**
```markdown
For frontend/backend communication outside of the initial page load context or form actions, use sveltekit remote functions
```

**After:**
```markdown
For API endpoints, follow the Axum router pattern with extractors and response types
```

### With Database → Without Database

**Before:**
```markdown
### Database Migration Workflow

When schema changes are required, follow this workflow:

1. **During development/iteration**: Use `npx prisma@6 db push`
2. **Final step**: Once finalized, create migration: `npx prisma@6 migrate dev --name description`
```

**After:**
```markdown
[Section removed - no database detected]
```

## Error Handling

**If project type cannot be determined:**
```
I couldn't detect your project type. Could you tell me:
- What language/framework are you using?
- How do you run tests?
- How do you lint/format code?
```

**If no test command found:**
```
I couldn't find a test command. Would you like to:
1. Provide one manually
2. Skip test-related sections
3. Add placeholder (e.g., "TODO: add tests")
```

**If plugin reference files are missing:**
```
Error: Reference templates not found in plugin directory.
Expected location: skills/setup/reference/
This indicates a plugin installation issue.
```

## Success Criteria

You've successfully completed this skill when:
- [ ] Project analysis is accurate and shown to user
- [ ] User preferences are captured
- [ ] All reference templates are read
- [ ] Each template is intelligently adapted (not just string-replaced)
- [ ] Files are written to .claude/ directory
- [ ] Summary is presented with next steps
- [ ] User understands how to use the new commands

The generated workflow should feel native to the project, not like a generic template that was forced to fit.
