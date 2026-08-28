# Upgrade Mode

Reference for the setup skill when an existing RPI installation is detected.

## Scope: only touch what this plugin generated

`.claude/skills/` and `.claude/agents/` are shared directories. A real project will have skills from other plugins and skills the team wrote themselves sitting right alongside the RPI ones — often many more of them than yours.

Your scope is exactly the files listed in Step 6 of `SKILL.md`, plus the retired files named in Step U4 below. Everything else in those directories is out of bounds: don't regenerate it, don't "tidy" it, don't remove it because it looks unfamiliar, and don't apply the residue rules below to it. A skill you don't recognize is almost certainly someone else's, not an old version of yours.

If a project skill happens to share a name with one of yours, stop and ask rather than overwriting.

The artifacts migration in Step U3 is the one part of an upgrade that reaches outside `.claude/`. It moves the workflow's own output directories and nothing else — see the constraints there.

## Step U1: Extract project adaptations

Read the existing commands/skills to recover the project-specific details, checking both `.claude/commands/` (v1) and `.claude/skills/` (v2+): test commands (unit, integration, e2e), lint/format/build/typecheck commands, database tooling and migration commands, framework-specific patterns, issue tracking integration, the artifacts directory the install writes to, and any custom additions the user made.

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

**Show only the deltas that apply to their installed version.** The block above is the v2→v3 story; don't show a v1 migration section to a v2 user, or the list above to someone already on v3. Take their version from `.claude/.rpi-version` (or the fallback ladder in `detection.md`), then read `${CLAUDE_PLUGIN_ROOT}/CHANGELOG.md` and compose the summary from the entries between their version and yours.

If you couldn't pin their version down, say so in the summary and show the union of the candidate ranges rather than picking one silently — an extra line about a change they already have is cheaper than not mentioning one they don't.

For an upgrade **from v4.1**, the only change is where artifacts live and what the agents that read them are called:

```
The default artifacts directory moved from thoughts/shared/ to .rpi/, and it's
flat — no subdirectory per type. The type is now the filename's last segment:

  thoughts/shared/plans/2026-01-05-auth.md  ->  .rpi/2026-01-05-auth-plan.md

A dated name keeps one feature's whole chain sorted together, and a hidden root
that only this workflow writes to can be gitignored in one line without stepping
on anything else in the repo.

You choose what happens to your install: keep thoughts/shared/ exactly as it is,
move to .rpi/, or name your own root. Moving relocates and renames the existing
artifacts and rewrites the links between them, so nothing goes stale.

  - thoughts-locator and thoughts-analyzer are now artifact-locator and
    artifact-analyzer. The names no longer assume a directory name, since the
    location is yours to pick. The old agent files are retired.
  - The two agents are now always generated. Previously they were conditional on
    a thoughts/ directory the workflow wrote to anyway.
  - Setup no longer asks how to structure the directory — the subdirectory names
    are fixed.

Nothing about how you invoke the skills changes.
```

For an upgrade **from v4.0**, the skill set is unchanged and only the handoffs between phases moved:

```
Same skills, same workflow. What changed is what a phase leaves behind when it
finishes — both files that outlive a phase now carry a per-phase record. This
matters if you start a fresh agent per phase; it's invisible if you don't.

  - Plans gained a ### Completion block per phase. /create-plan emits it empty,
    /implement-plan fills it in with deviations, waivers, and anything a later
    phase has to know. /iterate-plan won't touch a filled-in one.
  - Review metadata is now written incrementally — each phase appends its own
    section as it finishes, instead of one agent writing the whole file after
    the last phase. The per-file triage is now written by the agent that wrote
    the code, while it still has the reasoning.
  - The metadata file's name mirrors the plan's, so a fresh phase agent finds
    it without globbing.
  - /prepare-pr reads both, and knows to double-check any metadata section
    marked as reconstructed after the fact.

Nothing about how you invoke the skills changes. Plans already in flight have
no Completion blocks; the skills detect that and fall back rather than
retrofitting them mid-implementation.
```

For an upgrade **from v3**, the honest summary is that the workflow didn't change but the templates were rewritten:

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

## Step U3: Confirm the artifacts directory

`.rpi/` is the default as of v4.2 — hidden, flat, and owned entirely by this workflow, so nothing else in the repo lands in it. An existing install almost certainly writes to `thoughts/shared/` with a directory per type; `detection.md` tells you how to recover the root it actually uses. Never assume it.

Ask with `AskUserQuestion`:

- **Keep `<current root>`** — nothing moves. The regenerated skills keep writing where they write today. This is the zero-risk answer, and the right one for anyone mid-implementation or with the old paths wired into their own tooling.
- **Move to `.rpi/`** — adopt the new default. Existing artifacts move, get renamed to the flat convention, and the references between them are rewritten.
- **Somewhere else** — they name a root (`.output/`, `notes/`, whatever). Same migration, different destination.

Whatever they pick is the root you adapt every template against in Step U4. If they keep their current root, nothing moves and the rest of the upgrade proceeds unchanged. The flat naming is not optional either way — `/implement-plan` and `/prepare-pr` locate review metadata by swapping a plan's `-plan` suffix for `-review`.

### Migrating the artifacts

Only for the two answers that move files, and only before regenerating — the skills you write in Step U4 have to agree with what's on disk.

This is a move **and** a rename: the type used to be the directory, and now it's the filename's last segment.

```
thoughts/shared/plans/2026-01-05-auth.md   →   .rpi/2026-01-05-auth-plan.md
```

**Move only what this workflow owns:** `research/`, `designs/`, `plans/`, `review-metadata/`, and `tickets/` and `prs/` if they exist. Everything else under the old root belongs to the user — list it and ask before touching it.

1. **Move and rename each file.** The old default recommended gitignoring `thoughts/`, so most of these files are untracked and `git mv` will fail on them; use `git mv` where git tracks the file and plain `mv` where it doesn't. Never overwrite an existing destination — skip it and ask.

   ```bash
   mkdir -p .rpi
   for d in research designs plans review-metadata tickets prs; do
     case "$d" in
       research) t=research ;; designs) t=design ;; plans) t=plan ;;
       review-metadata) t=review ;; tickets) t=ticket ;; prs) t=pr ;;
     esac
     for f in "thoughts/shared/$d"/*.md "thoughts/shared/$d"/*.html; do
       [ -e "$f" ] || continue
       base=${f##*/}; dest=".rpi/${base%.*}-$t.${base##*.}"
       if [ -e "$dest" ]; then echo "SKIP $f — $dest exists, ask first"; continue; fi
       if git ls-files --error-unmatch "$f" >/dev/null 2>&1; then
         git mv "$f" "$dest"
       else
         mv "$f" "$dest"
       fi
     done
   done
   ```

   Only `.md` and `.html` move. If those directories hold anything else, say what you left behind.

2. **Rewrite the cross-references inside the moved files.** Plans link their design and research docs, review metadata links its plan, research docs link each other. Every one of those paths just changed shape, so a rewrite that only swaps the prefix leaves them broken:

   ```bash
   grep -rl 'thoughts/shared/' .rpi/ | while read -r f; do
     perl -pi -e 's{thoughts/shared/(research|designs|plans|review-metadata|tickets|prs)/([\w.-]+?)\.(md|html)}{".rpi/$2-" . {research=>"research",designs=>"design",plans=>"plan","review-metadata"=>"review",tickets=>"ticket",prs=>"pr"}->{$1} . ".$3"}ge' "$f"
     echo "rewrote $f"
   done
   ```

   Design mockups are `.html`, not just `.md`; the grep catches both. Report how many files it touched — zero means the move didn't land where you think it did. Then re-run the `grep -rl` to confirm nothing still points at the old tree.

3. **Update `.gitignore`.** Replace the old root's entry with `.rpi/` — one line, the whole directory. That's the point of a dedicated root: there's nothing else in it to protect.

4. **Clean up the old root.** If nothing is left under it, offer to remove it. If something is, leave it and say what's still there.

Substitute the chosen root for `.rpi/` throughout if they named their own. Nothing outside these directories moves: this is a migration of the workflow's own output, not a repo-wide path rewrite.

## Step U4: Regenerate

1. Read all reference templates (Step 4 of the main skill)
2. Adapt each one using the extracted details (see `adaptation.md`)
3. Write everything to `.claude/skills/` and `.claude/agents/`, including copying `reference/scripts/herdr-phase.sh` verbatim to `.claude/scripts/herdr-phase.sh` and `chmod +x`-ing it. Overwrite any existing copy so upgrades pick up script fixes. Refresh `.claude/.rpi-version` with the version you just generated from.
4. Clean up retired files, **asking first**:
   - `.claude/commands/{research-codebase,create-plan,iterate-plan,implement-plan}.md` — migrated to skills
   - `.claude/commands/read-ticket.md` — retired; `branch-ticket-detector` fetches tickets now
   - `.claude/skills/review-changes/` — retired, folded into `/prepare-pr`
   - `.claude/agents/{thoughts-locator,thoughts-analyzer}.md` — renamed to `artifact-locator` / `artifact-analyzer`. Carry any customization across into the new file before removing the old one, then check whether `/research-codebase` or `/iterate-plan` still name the old agents.
5. Handle the gitignore for the artifacts directory if it isn't configured yet
6. Show the summary and workflow tips (Steps 7-8 of the main skill)

## Step U5: Create the artifacts directory

```bash
mkdir -p .rpi
```

Nothing to create inside it — the artifacts are flat files named for their type.

## Preserving customizations

Users edit generated files. Before overwriting, look for content that doesn't match the templates — but "doesn't match" splits two ways, and telling them apart is the central judgment call of an upgrade:

**Theirs — preserve it.** Project-specific content: their commands, their conventions, guidance about their domain, whole sections you don't recognize from any template. Carry it into the regenerated file, or show it and ask. Silently discarding a user's customization is worse than a slightly awkward merge.

**Stale template — replace it.** Content that matches an *older* version of a template. The tell is repetition: if a workflow instruction appears more than once in the same file, that's almost always residue from a previous release, not a user deliberately emphasizing something. Collapse it to the current template's single statement.

When you genuinely can't tell, show the user the specific lines and ask. Guessing wrong in the "theirs" direction costs a little duplication; guessing wrong in the "stale" direction deletes their work.

If the user wants to be selective, show the list of files and let them choose which to regenerate.

### Upgrading from v3 or earlier

*(This section describes residue specific to v3-and-earlier installs. Delete it once those are no longer in circulation.)*

The current templates were rewritten for the Claude 5 generation of models — see "The register these templates are written in" in `adaptation.md` for the reasoning. The practical consequence for an upgrade is that v3 files carry a lot of duplication that reads like emphasis but isn't. Expect to find, and collapse:

- **The documentarian rule stated three times per agent** in `codebase-analyzer`, `codebase-locator`, and `codebase-pattern-finder` — an opening `CRITICAL` block, a "What NOT to Do" list, and a closing "you are a documentarian, not a critic" paragraph. The current template states it once, in the `description` plus one line of body.
- **A ~120-line invented pagination example** in `codebase-pattern-finder`, plus a malformed code fence that swallowed its own guidelines section.
- **`/design`'s "CRITICAL: THIS IS NOT A PLAN"** block of `DO NOT` lines, now a positive definition of what a design doc is.
- **`/implement-plan`'s "never use limit/offset"** instruction, removed — it fights the Read tool's own guidance.
- **`/iterate-plan`'s "Example Interaction Flows"** and subagent-spawning tutorial, both cut.
- **Joke descriptions** on `web-search-researcher` and `thoughts-analyzer` (now `artifact-analyzer`). Descriptions drive dispatch, so these were rewritten to describe what the agent actually does.
- **Hardcoded `npm` commands** in `/iterate-plan`'s success-criteria section, in a template that's supposed to be tooling-neutral. If the user's install has these and the project isn't Node, that's a bug to fix, not a customization to keep.

Anything else that diverges from the templates is far more likely to be theirs. Treat this list as exhaustive for "safe to replace without asking."
