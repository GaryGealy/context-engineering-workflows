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

**Show only the deltas that apply to their installed version.** The block above is the v2→v3 story; don't show a v1 migration section to a v2 user, or the list above to someone already on v3. `CHANGELOG.md` in the plugin root is the source of truth for what changed between any two versions — read it and compose the summary from the entries that apply.

For a **v3→v4** upgrade, the honest summary is that the workflow didn't change but the templates were rewritten:

```
Your skills and agents are the same set — nothing added or retired. What changed
is how the templates are written, for the Claude 5 generation of models:

  - Instructions that were repeated three times now appear once. Newer models
    don't need the emphasis, and duplicates cost them effort to reconcile.
  - Several "DO NOT" fences became plain definitions of what the artifact is.
  - The research agents lost a long invented example that anchored them to one
    way of looking; they now carry an output contract instead.
  - /design gained the ability to produce a concrete reference artifact — an
    HTML mockup, real payloads, a schema diff — that /create-plan and
    /implement-plan build against instead of prose.
  - /create-plan phases now name real test files and cases instead of
    "add tests" checkboxes.

Your project adaptations and any edits you made are preserved.
```

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

Users edit generated files. Before overwriting, look for content that doesn't match the templates — but "doesn't match" splits two ways, and telling them apart is the central judgment call of an upgrade:

**Theirs — preserve it.** Project-specific content: their commands, their conventions, guidance about their domain, whole sections you don't recognize from any template. Carry it into the regenerated file, or show it and ask. Silently discarding a user's customization is worse than a slightly awkward merge.

**Stale template — replace it.** Content that matches an *older* version of a template. The tell is repetition: if a workflow instruction appears more than once in the same file, that's almost always residue from a previous release, not a user deliberately emphasizing something. Collapse it to the current template's single statement.

When you genuinely can't tell, show the user the specific lines and ask. Guessing wrong in the "theirs" direction costs a little duplication; guessing wrong in the "stale" direction deletes their work.

If the user wants to be selective, show the list of files and let them choose which to regenerate.

### Upgrading from v3 or earlier

*(This section describes residue specific to v3-and-earlier installs. Delete it once those are no longer in circulation.)*

v4 rewrote the templates for the Claude 5 generation of models — see "The register these templates are written in" in `adaptation.md` for the reasoning. The practical consequence for an upgrade is that v3 files carry a lot of duplication that reads like emphasis but isn't. Expect to find, and collapse:

- **The documentarian rule stated three times per agent** in `codebase-analyzer`, `codebase-locator`, and `codebase-pattern-finder` — an opening `CRITICAL` block, a "What NOT to Do" list, and a closing "you are a documentarian, not a critic" paragraph. v4 states it once, in the `description` plus one line of body.
- **A ~120-line invented pagination example** in `codebase-pattern-finder`, plus a malformed code fence that swallowed its own guidelines section.
- **`/design`'s "CRITICAL: THIS IS NOT A PLAN"** block of `DO NOT` lines, now a positive definition of what a design doc is.
- **`/implement-plan`'s "never use limit/offset"** instruction, removed — it fights the Read tool's own guidance.
- **`/iterate-plan`'s "Example Interaction Flows"** and subagent-spawning tutorial, both cut.
- **Joke descriptions** on `web-search-researcher` and `thoughts-analyzer`. Descriptions drive dispatch, so these were rewritten to describe what the agent actually does.
- **Hardcoded `npm` commands** in `/iterate-plan`'s success-criteria section, in a template that's supposed to be tooling-neutral. If the user's install has these and the project isn't Node, that's a bug to fix, not a customization to keep.

Anything else that diverges from the templates is far more likely to be theirs. Treat this list as exhaustive for "safe to replace without asking."
