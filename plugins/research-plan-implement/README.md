# Research → Design → Plan → Implement → Review

**Context-aware workflow for AI-assisted development.**

Analyzes your codebase, aligns on design, creates detailed plans, implements features with automated verification, and guides structured review. Uses intentional compaction to manage context windows and maximize AI effectiveness.

## Installation

### Add the Marketplace

First, add this marketplace to Claude Code:

1. Run `/marketplace add`
2. Enter: `lucasnad27/claude-plugins`

### Install the Plugin

```bash
/plugin add lucasnad27/claude-plugins/research-plan-implement
```

### Upgrading

Re-run `/setup` in your project. It detects the installed version, shows what's changing, and asks which files to regenerate.

From v4.1 or earlier, it also asks where you want workflow artifacts to live: keep `thoughts/shared/`, move to the new `.rpi/` default, or name your own root. Moving relocates the artifacts, renames them to the flat convention, and rewrites the links between them.

From v1, note that generated files moved from `.claude/commands/` to `.claude/skills/`.

## Quick Start

1. Navigate to your project: `cd my-project`
2. Run setup: `/setup`
3. Learn the workflow: `/guide`
4. Start researching: `/research-codebase "How does auth work?"`
5. Align on design: `/design-doc .rpi/2026-04-02-auth-research.md`

## What You Get

**Skills:**
- `/guide` - Quick contextual orientation (where am I in the workflow? what's next?)
- `/setup` - Generate project-specific workflow commands

**Generated Skills** (after running `/setup`):
- `/research-codebase` - Research using parallel sub-agents, create research documents
- `/design-doc` - Align on design approach through collaborative discussion before planning
- `/create-plan` - Create detailed implementation plans through interactive research
- `/implement-plan` - Execute plans with automated verification and testing checkpoints
- `/prepare-pr` - Commit changes, open the PR, and land a numbered review guide: a short index in the description, the detail as inline review comments anchored to the diff
- `/guide` - Quick contextual orientation (where am I? what's next?)

**Generated Agents** (specialized AI assistants):
- `codebase-locator` - Find WHERE code lives (files, directories, components)
- `codebase-analyzer` - Analyze HOW code works (data flow, implementation details)
- `codebase-pattern-finder` - Find similar patterns and examples to model after
- `query-planner` - Decompose complex research questions into targeted sub-queries
- `branch-ticket-detector` - Detect the ticket from your branch/worktree so `/research-codebase` works with no arguments (if an issue tracker is configured)
- `web-search-researcher` - Research external docs and resources
- `artifact-locator` - Find prior research, designs, plans, and tickets under `.rpi/`
- `artifact-analyzer` - Extract decisions and constraints from one of those documents

**Generated Script:**
- `scripts/herdr-phase.sh` - If you run inside [herdr](https://herdr.dev), each workflow skill tags its tab with a phase emoji (🔬 research · 🎨 design · 📋 plan · 🔨 implement · 🔍 review) so the session sidebar becomes a phase board. Safe no-op outside herdr — nothing to configure. Run `/guide herdr` for details.

## Real-World Results

- **300k LOC Rust codebase:** 1-hour bug fix by non-expert, PR approved without revision
- **35k LOC feature:** 7 hours vs 3-5 days estimated, minimal PR revisions
- **Key insight:** Upfront research and design investment pays off exponentially

## How It Works

This plugin generates a complete "research → design → plan → implement → review" workflow in your project's `.claude/` directory by:

1. **Analyzing your project** - Reads your `package.json`, `Cargo.toml`, `go.mod`, or `pyproject.toml` to understand your stack
2. **Adapting intelligently** - Uses Claude's reasoning (not brittle templates) to customize commands for your tools
3. **Generating workflow** - Creates skills and agents that work natively with your project's build system

**No templates. No hardcoded rules.**

## Usage

### Quick Start

1. Navigate to your project directory
2. Run the setup skill:
   ```bash
   /setup
   ```
3. Answer a few questions about your preferences
4. Start using the generated skills!

### What Gets Generated

The plugin creates this structure in your project:

```
.claude/
├── skills/
│   ├── research-codebase/
│   │   └── SKILL.md
│   ├── design-doc/
│   │   └── SKILL.md
│   ├── create-plan/
│   │   └── SKILL.md
│   ├── implement-plan/
│   │   └── SKILL.md
│   ├── prepare-pr/
│   │   └── SKILL.md
│   └── guide/
│       └── SKILL.md
└── agents/
    ├── codebase-analyzer.md
    ├── codebase-locator.md
    ├── codebase-pattern-finder.md
    ├── query-planner.md
    ├── branch-ticket-detector.md # If an issue tracker is configured
    ├── artifact-analyzer.md
    ├── artifact-locator.md
    └── web-search-researcher.md
```

The skills write their artifacts to `.rpi/` by default — flat, with the type as the filename's last segment:

```
.rpi/
├── 2026-01-05-auth-research.md    # /research-codebase
├── 2026-01-05-auth-design.md      # /design-doc
├── 2026-01-05-auth-design.html    #   ...and its mockup, when the work has a shape
├── 2026-01-05-auth-plan.md        # /create-plan
└── 2026-01-05-auth-review.md      # /implement-plan
```

A dated name keeps one feature's whole chain sorted together, and a directory only this workflow writes to takes a one-line `.gitignore` entry without stepping on anything else in the repo. Setup asks before settling on a root, so `.output/`, `notes/`, or anything else works — the naming convention stays either way, since `/prepare-pr` finds a plan's review metadata by swapping `-plan` for `-review`.

### Example: TypeScript/SvelteKit Project

**Before running setup:**

```json
// package.json
{
  "scripts": {
    "test:unit": "vitest run",
    "lint": "eslint .",
    "format": "prettier --write .",
    "build": "vite build"
  }
}
```

**After running setup:**

Generated skills will use your actual scripts:

- Tests: `npm run test:unit` (not generic `npm test`)
- Linting: `npm run lint`
- Formatting: `npm run format`
- Build: `npm run build`
- Database: `npx prisma@6 db push` (if Prisma detected)

### Example: Rust Project

**Before running setup:**

```toml
# Cargo.toml
[package]
name = "my-api"
```

**After running setup:**

Generated skills will use Rust tooling:

- Tests: `cargo test`
- Linting: `cargo clippy`
- Formatting: `cargo fmt`
- Build: `cargo build`

### Example: Python/Django Project

**Before running setup:**

```toml
# pyproject.toml
[tool.poetry]
dependencies = { django = "^4.0" }
```

**After running setup:**

Generated skills will use Django patterns:

- Tests: `pytest tests/unit`
- Linting: `ruff check .`
- Formatting: `black .`
- Migrations: `python manage.py migrate`

## Understanding the Workflow

This plugin implements **intentional compaction**—a strategy for managing AI agent context windows by distilling progress into structured artifacts (research docs, design docs, plans) before starting fresh contexts.

**Why it matters:** Your context window is your ONLY lever to affect output quality without retraining models.

### The Five Phases

1. **Research** - Explore codebase without polluting main context
2. **Design** - Align on approach before committing to an implementation path
3. **Plan** - Create exact implementation specification
4. **Implement** - Execute phase-by-phase with testing-aware verification
5. **Review** - Commit, open the PR, and land a numbered review guide as inline stops on the diff

**Run `/guide` to learn how to use this workflow effectively.**

## Typical Workflow

### 1. Research the Codebase

```bash
/research-codebase "How does user authentication work?"
```

This spawns parallel agents to:

- Locate auth-related files
- Analyze how authentication is implemented
- Find usage patterns and examples
- Create a research document at `.rpi/*-research.md`

### 2. Align on Design

```bash
/design-doc .rpi/2026-04-02-auth-research.md
```

This:

- Reviews the research document
- Asks clarifying questions about approach and constraints
- Explores tradeoffs between implementation options
- Produces a concrete reference artifact where the work has a shape worth rendering — a self-contained HTML mockup for UI work, real request/response payloads for an API, a schema diff for a data model change
- Creates a design doc at `.rpi/*-design.md` that the plan will reference

### 3. Create Implementation Plan

```bash
/create-plan .rpi/add-oauth-support-ticket.md
```

This:

- Reads the ticket and any referenced design docs
- Researches relevant code patterns
- Asks clarifying questions
- Creates detailed plan at `.rpi/*-plan.md`

### 4. Revise the Plan (as needed)

Edit the plan file directly — there's no separate command. Keep it internally consistent, and
leave a completed phase's `### Completion` block alone; it's a record of what that phase did,
and a later phase reads it as its only memory of the earlier one.

### 5. Implement the Plan

```bash
/implement-plan .rpi/2025-01-05-add-oauth-plan.md
```

This:

- Reads the plan
- Implements each phase with testing in mind from the start
- Runs automated verification (tests, linting, builds)
- Pauses for manual testing between phases
- Updates checkboxes in the plan as progress is made

### 6. Prepare PR

```bash
/prepare-pr            # commit, push, open a PR for the current branch
/prepare-pr 123        # update the description of existing PR #123
```

This:

- Commits any outstanding changes and pushes the branch
- Analyzes the diff (current branch vs the default branch)
- Builds a numbered list of **stops** — each a file, a line or range, a type (`issue` / `note` / `suggestion` / `yagni`), and a claim to test
- Posts them as inline review comments, so each stop is a resolvable thread a reviewer ticks off
- Puts a numbered index in the PR description, capped at 60 lines, plus test coverage and anything deferred
- Optionally walks the stops with you in [tuicr](https://github.com/agavra/tuicr), where the posted threads render natively
- `--no-stops` keeps the whole guide in the description; that's also the default on forges without inline review comments
- Can instead point at an existing PR number and write a guide onto it

## Supported Project Types

Currently adapts intelligently to:

### Languages

- TypeScript/JavaScript (Node.js, Deno, Bun)
- Python
- Go
- Rust

### Frameworks

- SvelteKit
- Next.js
- Django
- FastAPI
- Generic frameworks (with sensible defaults)

### Build Systems

- npm/yarn/pnpm scripts
- Makefile
- Cargo
- Poetry
- Go modules

### Databases

- Prisma
- Drizzle
- SQLAlchemy
- Django ORM
- Diesel

## Philosophy

This plugin generates workflows that follow these principles:

1. **Documentarian Approach** - Research and document what EXISTS, not what SHOULD BE
2. **Design Before Planning** - Align on approach before committing to an implementation path
3. **Parallel Sub-Agents** - Spawn specialized agents concurrently for efficiency
4. **Interactive Planning** - Iterative, collaborative plan creation with user feedback
5. **Testing-Aware Implementation** - Tests are not an afterthought; they're built into each phase
6. **Automated + Manual Verification** - Clear separation of what can be automated vs requires human testing
7. **The Smallest Thing That Works** - Check for an existing helper, the stdlib, then an installed dependency before specifying new code; a one-caller abstraction is a reviewable finding
8. **Comments Default To None** - Rationale lives in the design doc and the plan, not narrated into the source
9. **Review Is Navigation, Not A Verdict** - The PR carries numbered stops naming what to decide, anchored to the lines they're about

## Customization

### After Generation

All generated files are standard markdown in `.claude/` - you can edit them freely:

- Add project-specific guidance
- Customize success criteria
- Add more agents
- Modify workflows

### Preserving Customizations

When you re-run `/setup`, it will:

1. Detect existing `.claude/` files
2. Ask which files to regenerate
3. Preserve your custom sections

### Sharing with Team

Commit `.claude/` to version control so your team gets the same workflow:

```bash
git add .claude/
git commit -m "Add research/design/plan/implement/review workflow"
git push
```

## Troubleshooting

### "I couldn't detect your project type"

The plugin looks for:

- `package.json` (Node/TypeScript)
- `Cargo.toml` (Rust)
- `go.mod` (Go)
- `pyproject.toml` or `requirements.txt` (Python)

If none exist, it will ask you to manually specify your stack.

### "Reference templates not found"

This means the plugin isn't installed correctly. Ensure:

1. Plugin is in Claude's plugins directory
2. `skills/setup/reference/` directory exists
3. Reference templates are present

### Generated skills don't match my project

The plugin adapts based on what it finds in config files. If it gets something wrong:

1. Re-run `/setup` with correct info
2. Manually edit the generated `.claude/` files
3. File an issue so we can improve detection

### My artifacts are still in `thoughts/shared/`

Nothing breaks — an upgrade only moves them if you ask it to. Re-run `/setup` and pick `.rpi/` (or your own root) when it asks; it relocates the files, renames them to the flat convention, and rewrites the links between them. Picking "keep what I have" is equally supported, and the generated skills keep writing where they do today.

### Upgrading from v1 to v2

Generated files moved from `.claude/commands/` to `.claude/skills/`. Re-run `/setup` to generate the new skill files. You can safely delete the old `commands/` files once the new skills are confirmed working.

## Examples

### Research Example

```bash
/research-codebase "How do we handle database migrations?"
```

**Output:**

- Research document at `.rpi/2026-04-02-database-migrations-research.md`
- Includes file references, code examples, and architecture notes
- Documents current state without recommendations

### Design Example

```bash
/design-doc .rpi/2026-04-02-database-migrations-research.md
```

**Process:**

1. Reviews existing research
2. Asks about constraints (downtime tolerance, rollback requirements, etc.)
3. Explores migration strategy options
4. Creates design doc at `.rpi/2026-04-02-migration-strategy-design.md`

### Planning Example

```bash
/create-plan "Add two-factor authentication"
```

**Process:**

1. Asks clarifying questions
2. Researches existing auth code
3. Proposes implementation phases
4. Creates plan at `.rpi/2026-04-02-add-2fa-plan.md`

### Implementation Example

```bash
/implement-plan .rpi/2026-04-02-add-2fa-plan.md
```

**Process:**

1. Reads plan
2. Implements Phase 1 with tests alongside code
3. Runs tests: `npm run test:unit`
4. Runs linting: `npm run lint`
5. Pauses for manual testing
6. Continues to Phase 2 after confirmation

### Prepare PR Example

```bash
/prepare-pr
```

**Process:**

1. Commits any outstanding changes and pushes the branch
2. Diffs current branch against main
3. Builds one numbered list of stops — a file, a line, a type, and a claim to test
4. Maps test coverage across the changes
5. Opens the PR, posts the stops as inline review comments, and puts a numbered index in the description
6. Optionally walks the stops with you in tuicr
7. Or, given a PR number, writes the guide onto that existing PR instead

## Contributing

Contributions welcome! Areas we'd love help with:

- Additional language support (Java, C#, PHP, etc.)
- Framework-specific guidance improvements
- Better project detection heuristics
- Documentation improvements

## Attribution

This workflow is inspired by and adapted from multiple sources in the AI-assisted development community.

**Primary inspiration:**
- **HumanLayer** - Original research → plan → implement pattern and intentional compaction strategy
  - **Website:** [humanlayer.dev](https://humanlayer.dev)
  - **GitHub:** [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
  - **AI Engineering Talk:** [YouTube](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i) - Deep dive on context engineering for coding agents
  - [Advanced Context Engineering for Coding Agents](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents) - Detailed guide on the principles behind this workflow

**Additional influences:**
- **CRISPY / Dex** - Design-before-planning discipline and structured review phases
- **Simon Willison** - Practical AI-assisted development patterns and the value of explicit workflow documentation
- **[ponytail](https://github.com/DietrichGebert/ponytail)** (MIT, by DietrichGebert) - The reuse-before-writing ladder in `/create-plan`, root-cause-over-symptom in `/implement-plan`, and treating a one-caller abstraction as a reviewable finding (the `yagni` stop type in `/prepare-pr`)

The intentional compaction strategy and multi-phase workflow originated from HumanLayer's work on optimizing AI agent effectiveness through context window management, expanded with design alignment and review phases drawn from the broader AI engineering community.

## License

MIT
