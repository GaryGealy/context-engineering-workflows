# Research → Design → Plan → Implement → Review

**Context-aware workflow for AI-assisted development.**

Analyzes your codebase, aligns on design, creates detailed plans, implements features with automated verification, and guides structured review. Uses intentional compaction to manage context windows and maximize AI effectiveness.

## Installation

This plugin is a **generator**. You install it once per machine, then run `/setup` in each project to produce that project's workflow files. Those files get committed, so teammates install nothing — see [For your teammates](#for-your-teammates).

### Claude Code

Add the marketplace:

```
/plugin marketplace add GaryGealy/context-engineering-workflows
```

Then install the plugin:

```
/plugin install research-plan-implement@research-plan-implement-workflow
```

### VS Code (Copilot chat)

VS Code's Agent Plugins resolver reads `.claude-plugin/marketplace.json` as one of its four recognized manifest formats, so this repo installs as-is — there's no separate VS Code package.

1. `Cmd/Ctrl+Shift+P` → **Chat: Install Plugin From Source**
2. Enter `GaryGealy/context-engineering-workflows`
3. Turn on agent skills in your settings:

   ```json
   { "chat.useAgentSkills": true }
   ```

Requires VS Code 1.108 or newer; on older builds the setting is named `github.copilot.chat.skillTool.enabled`.

To browse it alongside your other plugins, open the Extensions view and filter by `@agentPlugins`. To subscribe to the whole marketplace rather than one plugin, add the repo to `chat.plugins.marketplaces` (or run **Chat: Manage Plugin Marketplaces**).

### Then, in your project

```bash
cd my-project
```

```
/setup     # generates this project's workflow files
/guide     # orientation, any time
```

Name VS Code as a target during `/setup` and it also merges `chat.useAgentSkills` into `.vscode/settings.json` for you.

### For your teammates

They don't install the plugin. `/setup` writes real files into the repository:

```
.claude/skills/         # the phase commands
.claude/agents/         # the research subagents
.claude/scripts/        # herdr phase markers
.vscode/settings.json   # chat.useAgentSkills, if VS Code is a target
```

Commit those and anyone who clones has the workflow. Claude Code picks it up automatically; VS Code picks it up too, since it scans `.claude/skills/` and `.claude/agents/` natively and the committed `.vscode/settings.json` supplies the setting. Run `/guide copilot` for what VS Code maps and what it drops.

### Upgrading from v1

If you installed v1 of this plugin, re-run `/setup` in your project to upgrade your generated workflow files to v2. The setup skill will detect existing files and ask which to regenerate.

Note: v2 moves generated files from `.claude/commands/` to `.claude/skills/` to align with Claude Code's current conventions. Re-running `/setup` will create the new skill files alongside (or in place of) the old command files.

## Quick Start

1. Navigate to your project: `cd my-project`
2. Run setup: `/setup`
3. Learn the workflow: `/guide`
4. Start researching: `/research-codebase "How does auth work?"`
5. Align on design: `/design-doc thoughts/shared/research/2026-04-02-auth.md`

## What You Get

**Skills:**
- `/guide` - Quick contextual orientation (where am I in the workflow? what's next?)
- `/setup` - Generate project-specific workflow commands

**Generated Skills** (after running `/setup`):
- `/research-codebase` - Research using parallel sub-agents, create research documents
- `/design-doc` - Align on design approach through collaborative discussion before planning
- `/create-plan` - Create detailed implementation plans through interactive research
- `/iterate-plan` - Update plans based on feedback or new discoveries
- `/implement-plan` - Execute plans with automated verification and testing checkpoints
- `/prepare-pr` - Commit changes, open the PR, and write its description as a structured review guide (or update an existing PR's description)
- `/guide` - Quick contextual orientation (where am I? what's next?)

**Generated Agents** (specialized AI assistants):
- `codebase-locator` - Find WHERE code lives (files, directories, components)
- `codebase-analyzer` - Analyze HOW code works (data flow, implementation details)
- `codebase-pattern-finder` - Find similar patterns and examples to model after
- `query-planner` - Decompose complex research questions into targeted sub-queries
- `branch-ticket-detector` - Detect the ticket from your branch/worktree so `/research-codebase` works with no arguments (if an issue tracker is configured)
- `web-search-researcher` - Research external docs and resources
- `thoughts-locator` - Find documents in thoughts/ directory (optional)
- `thoughts-analyzer` - Extract insights from thought documents (optional)

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
│   ├── design/
│   │   └── SKILL.md
│   ├── create-plan/
│   │   └── SKILL.md
│   ├── iterate-plan/
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
    ├── thoughts-analyzer.md      # If thoughts/ enabled
    ├── thoughts-locator.md       # If thoughts/ enabled
    └── web-search-researcher.md
```

### Editors

The generated files run in **Claude Code** and **VS Code Copilot chat** from a single copy — VS Code scans `.claude/skills/` and `.claude/agents/` alongside its own `.github/` equivalents. One difference is load-bearing, so `/setup` asks which editors you use.

**Skills generated for VS Code carry no `model:` or `effort:` frontmatter.** A skill whose frontmatter includes `model:` hangs Copilot chat when invoked — no output, no error, and the session stays dead until VS Code restarts. The key does it regardless of value; VS Code's SKILL.md spec documents only `name`, `description`, `argument-hint`, `user-invocable`, and `disable-model-invocation`. Setup strips both fields when Copilot is a target, and the upgrade path strips them from installs generated by 4.2.0 or earlier.

The tradeoff is per-skill model pinning. Claude Code-only installs keep it — research and planning on opus at high effort, `/guide` on haiku. In Copilot every skill runs on the model selected in chat, so choose it before starting a research or planning pass.

Copilot may also need one setting, which `/setup` merges into `.vscode/settings.json` when you name it as a target:

```json
{ "chat.useAgentSkills": true }
```

Current VS Code documents `chat.agentSkillsLocations` instead and lists `.claude/skills` among its defaults, with skills on by default since 1.109 — so on a current build this may be a no-op. It is harmless, and kept for older ones.

Type `/` in the Copilot chat input and the phase skills appear as slash commands. Run `/guide copilot` for the full picture, including what remains unverified about tool-name translation.

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
5. **Review** - Commit, open the PR, and write its description as a structured review guide

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
- Create a research document in `thoughts/shared/research/`

### 2. Align on Design

```bash
/design-doc thoughts/shared/research/2026-04-02-auth-research.md
```

This:

- Reviews the research document
- Asks clarifying questions about approach and constraints
- Explores tradeoffs between implementation options
- Produces a concrete reference artifact where the work has a shape worth rendering — a self-contained HTML mockup for UI work, real request/response payloads for an API, a schema diff for a data model change
- Creates a design doc in `thoughts/shared/designs/` that the plan will reference

### 3. Create Implementation Plan

```bash
/create-plan thoughts/tickets/add-oauth-support.md
```

This:

- Reads the ticket and any referenced design docs
- Researches relevant code patterns
- Asks clarifying questions
- Creates detailed plan in `thoughts/shared/plans/`

### 4. Iterate on Plan

```bash
/iterate-plan thoughts/shared/plans/2025-01-05-add-oauth.md
```

Update the plan based on feedback, new discoveries, or changed requirements.

### 5. Implement the Plan

```bash
/implement-plan thoughts/shared/plans/2025-01-05-add-oauth.md
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
- Analyzes the diff (current branch vs main)
- Categorizes changes as critical vs mechanical
- Maps test coverage across changed files
- Opens the PR with a description written as a structured review guide (suggested reading order)
- Can instead point at an existing PR number and rewrite its description

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

### Upgrading from v1 to v2

Generated files moved from `.claude/commands/` to `.claude/skills/`. Re-run `/setup` to generate the new skill files. You can safely delete the old `commands/` files once the new skills are confirmed working.

## Examples

### Research Example

```bash
/research-codebase "How do we handle database migrations?"
```

**Output:**

- Research document at `thoughts/shared/research/2026-04-02-database-migrations.md`
- Includes file references, code examples, and architecture notes
- Documents current state without recommendations

### Design Example

```bash
/design-doc thoughts/shared/research/2026-04-02-database-migrations.md
```

**Process:**

1. Reviews existing research
2. Asks about constraints (downtime tolerance, rollback requirements, etc.)
3. Explores migration strategy options
4. Creates design doc at `thoughts/shared/designs/2026-04-02-migration-strategy.md`

### Planning Example

```bash
/create-plan "Add two-factor authentication"
```

**Process:**

1. Asks clarifying questions
2. Researches existing auth code
3. Proposes implementation phases
4. Creates plan at `thoughts/shared/plans/2026-04-02-add-2fa.md`

### Implementation Example

```bash
/implement-plan thoughts/shared/plans/2026-04-02-add-2fa.md
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
3. Categorizes each changed file (critical / mechanical / tests)
4. Maps test coverage across the changes
5. Opens the PR with a description written as a review guide (suggested reading order)
6. Or, given a PR number, rewrites that existing PR's description instead

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

The intentional compaction strategy and multi-phase workflow originated from HumanLayer's work on optimizing AI agent effectiveness through context window management, expanded with design alignment and review phases drawn from the broader AI engineering community.

## License

MIT
