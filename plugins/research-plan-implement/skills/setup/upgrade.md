# Upgrade Mode

Reference for the setup skill when an existing RPI installation is detected.

## Step U1: Extract project adaptations

Read the existing commands/skills to recover the project-specific details, checking both `.claude/commands/` (v1) and `.claude/skills/` (v2+): test commands (unit, integration, e2e), lint/format/build/typecheck commands, database tooling and migration commands, framework-specific patterns, issue tracking integration, thoughts directory configuration, and any custom additions the user made.

These are the user's answers from their original setup. Reuse them rather than re-interrogating the project.

## Step U2: Show the upgrade summary

Tell the user what's changing before touching anything:

```
Detected existing RPI installation. Here's what's changing:

MIGRATION (v1 only):
  Commands are moving from .claude/commands/ to .claude/skills/
  Old command files will be removed after migration

NEW skills:
  /design — Lightweight design discussion before planning (~200 lines vs ~1000 line plans)
  /prepare-pr — Commit, open the PR, and write its description as a review guide
  /guide — Contextual orientation (where am I? what's next?)

RETIRED skills:
  /review-changes — Folded into /prepare-pr; the old skill directory will be removed

NEW agents:
  query-planner — Keeps research objective by separating questions from intent
  branch-ticket-detector — Detects the ticket from your branch so /research-codebase
    works with no arguments (only if an issue tracker is configured)

NEW script:
  scripts/herdr-phase.sh — Tags each tab with its workflow phase (🔬 🎨 📋 🔨 🔍)
    in the herdr sidebar. No-op outside herdr.

UPDATED skills:
  /research-codebase — Query planning keeps research objective; auto-detects the
    ticket from your branch when run without arguments
  /create-plan — Slimmed down (design decisions moved to /design), vertical phases
  /implement-plan — Testing-aware (TDD/conformance/manual), generates review metadata

UNCHANGED:
  /iterate-plan — Content unchanged

Your project adaptations will be preserved:
  - Test command: [extracted]
  - Lint command: [extracted]
  - [etc.]

Ready to upgrade? (yes / let me see details for a specific skill)
```

Adjust the summary to what's actually changing for *this* installation — don't list a migration section for a v2 user.

## Step U3: Regenerate

1. Read all reference templates (Step 4 of the main skill)
2. Adapt each one using the extracted details (see `adaptation.md`)
3. Write everything to `.claude/skills/` and `.claude/agents/`, including copying `reference/scripts/herdr-phase.sh` verbatim to `.claude/scripts/herdr-phase.sh` and `chmod +x`-ing it. Overwrite any existing copy so upgrades pick up script fixes.
4. Clean up retired files, **asking first**:
   - `.claude/commands/{research-codebase,create-plan,iterate-plan,implement-plan,read-ticket}.md` — migrated to skills
   - `.claude/skills/review-changes/` — retired, folded into `/prepare-pr`
5. Handle the thoughts gitignore if it isn't configured yet
6. Show the summary and workflow tips (Steps 7-8 of the main skill)

## Step U4: Create missing directories

If `thoughts/shared/` exists but the newer subdirectories don't:

```bash
mkdir -p thoughts/shared/designs thoughts/shared/review-metadata
```

## Preserving customizations

Users edit generated files. Before overwriting, look for sections that don't match the templates — those are theirs. Carry them into the regenerated file, or show them and ask. Silently discarding a user's customization is worse than a slightly awkward merge.

If the user wants to be selective, show the list of files and let them choose which to regenerate.
