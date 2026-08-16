# Changelog

All notable changes to the `research-plan-implement` plugin are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com).

## [4.1.0] - 2026-08-15

The `### Completion` block and the incremental review metadata below come from
one gap: the templates assumed a single agent spanning every phase of a plan,
accumulating notes in its own context. Running a fresh agent per phase — which
is the point of the workflow — deletes that context at every boundary. Anything
a later phase or `/prepare-pr` needs now has to be on disk before a phase stops,
so both files that outlive a phase gained a per-phase record.

### Added

- Plans now carry a `### Completion` block per phase. `/create-plan` emits it
  empty; `/implement-plan` fills it in before the pause message with the
  phase's deviations, anything the user waived or left unproven, and anything a
  later phase has to account for. The plan is the one file every phase agent
  reads, so it's where a finished phase leaves what the next one needs.
  - `/iterate-plan` carries filled-in blocks across intact and never edits
    them — they're the record of a phase whose author has already exited
  - `/prepare-pr` reads them for the deviations and waivers that belong in the
    PR description
  - Splits cleanly from the review metadata by audience: the completion block
    is what changed relative to the plan, the metadata is per-file review
    triage

### Changed

- `/implement-plan` now builds review metadata **incrementally, one section per
  phase**, instead of writing it once after the last phase. Under the old
  design a phase-5 agent had to re-derive the per-file triage by reading a diff
  it never wrote — which is the exact cost the metadata existed to eliminate,
  just moved from `/prepare-pr` to the last phase:
  - The metadata file's basename now **mirrors the plan's**, so an agent with no
    memory of earlier phases finds it in one Read instead of globbing a
    directory by date
  - Each phase appends a `## Phase N` section covering only the files it
    touched — Needs careful review / Mechanical / Tests / Deliberate non-fixes —
    written while the reasoning is still in context, before the pause message
  - The last phase adds a `## Summary` for the cross-cutting reads no single
    phase owns: what to open first, what's unproven across the whole change,
    what a later phase superseded
  - A section reconstructed after the fact is headed `(reconstructed from the
    diff — not authored in-phase)`, so downstream readers can tell author-grade
    triage from reader-grade
  - `review-metadata-template.md` reshaped from one flat document into the
    per-phase sections, with a back-link to the plan it belongs to
- `/prepare-pr` reads the metadata by mirroring the plan's basename rather than
  matching recent dates, and treats reconstructed sections as claims to verify
  against the diff rather than author intent to repeat. It also reads the plan
  itself now, not just the design doc.
- `/implement-plan`'s review metadata is no longer described as optional, and is
  written silently — it's plumbing between two skills, not a deliverable, and
  the user shouldn't have to decide about it on every phase.

### Fixed

- `herdr-phase.sh` stamped the phase glyph onto the **focused** tab rather than
  the agent's own tab, so in a multi-tab workspace the agent's label went stale
  while a sibling tab (often a human-run orchestrator) collected a stray prefix
  that later runs couldn't strip. The script now resolves its tab from
  `$HERDR_TAB_ID`, which herdr exports into each pane, and keeps the
  focused-pane scan only as a fallback. Existing installs carry a copy of this
  script at `.claude/scripts/herdr-phase.sh` — re-run `/setup` to pick up the
  fix, and clean up any stacked prefixes by hand with
  `herdr tab rename <id> "<label>"`. ([#15](https://github.com/lucasnad27/claude-plugins/issues/15))
- `/implement-plan` gave contradictory instructions about small plan deviations,
  telling the agent both to record them in the review metadata and to leave them
  to the plan. Deviations now go wherever they aren't already: cross-referenced
  when the plan carries per-phase completion blocks, recorded in the metadata
  when it doesn't.

## [4.0.0] - 2026-07-27

### Changed

- Rewrote the agent and skill templates for the Claude 5 generation of models,
  following Anthropic's [context engineering
  guidance](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models).
  Newer models infer intent well enough that the old guardrails cost more than
  they bought:
  - Collapsed the triplicated "documentarian" prohibition blocks in
    `codebase-analyzer`, `codebase-locator`, `codebase-pattern-finder`, and
    `research-codebase` down to a single statement each, folded into the
    `description` so it also improves dispatch
  - Resolved four instruction conflicts, including `codebase-pattern-finder`
    being told both to note the preferred pattern and never to recommend one
  - Replaced the 120-line invented pagination example in
    `codebase-pattern-finder` with an output contract — the example was
    JavaScript in an agent that runs against Rust, Go, and Python repos
  - Split the 745-line `setup` skill into a routing spine plus `detection.md`,
    `adaptation.md`, and `upgrade.md`, loaded only on the path that needs them
  - Restated `/design`'s three `DO NOT` lines as a definition of what a design
    doc is — same boundary, no fence
  - Slimmed `/iterate-plan` (276 → 117 lines) by cutting worked interaction
    examples and a subagent-spawning tutorial the agent descriptions cover
  - Rewrote the `web-search-researcher`, `thoughts-analyzer`, and
    `thoughts-locator` descriptions, which were jokes; descriptions drive
    dispatch and count against the skill-listing character cap
  - Dropped `/implement-plan`'s "never use limit/offset" instruction, which
    fought the Read tool's own guidance
- `/setup` now carries the reasoning behind the template style, so an agent
  regenerating someone's skills understands what it's preserving rather than
  copying shapes:
  - `adaptation.md` explains the register the templates are written in — one
    statement per constraint, definitions over prohibitions, and which
    prohibitions deliberately remain — with a length check against the source
    template to catch re-explanation creeping back in
  - `upgrade.md` gives a decision rule for the hard call in any upgrade:
    project-specific content is the user's and must survive, while an
    instruction repeated within a file is stale template and should collapse
  - `upgrade.md` lists the v3-and-earlier residue that is safe to replace
    without asking, scoped so it can be deleted once those installs age out
  - The upgrade summary is now composed from `CHANGELOG.md` for the user's
    actual version delta, instead of always showing the v2→v3 story
- `/design` now produces a concrete reference artifact — a self-contained HTML
  mockup for UI work, real payloads for an API, a schema diff for data model
  changes — and `/create-plan` and `/implement-plan` build against it rather
  than against prose describing it. `/guide design` describes the artifact as
  part of a good design, rather than counting code snippets against one
- `/create-plan` phases now specify test files and named test cases instead of
  "add tests for X" checkboxes
- `/guide tips` no longer pins its closing section to a specific model release.
  It had gone stale twice, and most of what it said ("give complete context
  upfront", "`/implement-plan` is the auto-mode candidate") describes the
  workflow rather than any one model. The durable advice stays under a
  model-neutral heading; the release-specific steering phrases and effort
  defaults are gone, since `effort:` lives in each skill's frontmatter anyway

### Fixed

- `codebase-pattern-finder` had a malformed code fence that rendered its own
  operating guidelines (Pattern Categories, Important Guidelines, What NOT to
  Do) inside a code block
- `/setup` listed a `read-ticket` skill and a `ticket-reader` agent in its
  output tree that no reference template ever backed, and `/research-codebase`
  pointed at "the project's ticket-reading agent" to match. Both dropped;
  `branch-ticket-detector` already fetches ticket contents, and a one-off
  lookup of a related ticket doesn't need a subagent

### Added

- herdr phase markers: each workflow skill tags its herdr tab with an emoji
  prefix (🔬 research · 🎨 design · 📋 plan · 🔨 implement · 🔍 review) so the
  session sidebar doubles as a phase board. Backed by a copied-verbatim
  `scripts/herdr-phase.sh` that no-ops outside herdr, so it's harmless for
  projects whose author doesn't use herdr
- `/guide herdr` topic explaining the phase markers and the manual override
- `.claude/.rpi-version`, written on every install and upgrade, so `/setup` can
  tell which version generated a user's files. The changelog-driven upgrade
  summary needs their version to pick the right entries, and nothing recorded
  it before. Installs predating this fall back to inferring the major from the
  file set — `/prepare-pr` means 3.x, `/review-changes` means 2.x

## [3.0.0] - 2026-06-03

### Added

- `/prepare-pr` skill that bundles the change review and pull-request
  preparation into a single guided flow
- `branch-ticket-detector` agent that infers the associated ticket from the
  current branch name
- `/research-codebase` now auto-detects the ticket from the branch and folds
  it into the research context

### Removed

- `/review-changes` skill, replaced by `/prepare-pr`

## [2.1.1] - 2026-04-21

### Fixed

- Scoped the `thoughts-locator` agent template to the current repo's
  `thoughts/` directory only. Previously it ranged across parent directories,
  sibling worktrees, and `~/thoughts`, causing slow searches and excessive
  permission prompts. Also dropped unused references to `thoughts/searchable/`,
  `thoughts/global/`, and per-user subdirs in the `research-codebase` skill.

## [2.1.0] - 2026-04-16

Optimizations for Claude Opus 4.7.

### Changed

- Bumped reasoning effort to `xhigh` for agentic skills (`research-codebase`,
  `create-plan`, `implement-plan`, `iterate-plan`, `design`, `review-changes`)
  to take advantage of Opus 4.7's extended thinking
- Refined skill guidance across `create-plan`, `implement-plan`,
  `research-codebase`, `iterate-plan`, `design`, and `review-changes` for
  Opus 4.7
- Updated `codebase-analyzer` and `thoughts-analyzer` agent prompts
- Expanded setup skill and `/guide` topics

### Fixed

- Corrected `AskUserQuestion` batching guidance

## [2.0.0] - 2026-04-05

Major rewrite migrating the workflow to the skills format, with new alignment
and review stages and testing-aware planning/implementation.

### Added

- `/design` skill for lightweight human-agent alignment before planning
- `/review-changes` skill for structured, guided code review
- `/guide` skill providing contextual workflow orientation (consolidates the
  former `/guide` and `/workflow-guide` into a single entry point)
- `query-planner` agent for objective research decomposition
- Vertical phase planning with per-phase testing in `create-plan`
- Testing-aware implementation and review metadata in `implement-plan`
- Pattern-finder now discovers existing testing infrastructure
- Upgrade intelligence and skills migration support in the setup skill
- Backtick injection in `/guide` for instant workspace state

### Changed

- Migrated `create-plan`, `iterate-plan`, and `implement-plan` from the
  commands format to the skills format
- Rewrote vertical phase guidance with concrete examples
- Updated README and workflow docs to reflect the 5-step flow
- Output templates extracted into dedicated supporting files
- Research/plan commands now use the `AskUserQuestion` tool for open questions
- Setup skill now recommends creating a branch before running

### Fixed

- README inconsistencies surfaced in review
- Missing space between `/review-changes` command and its filename argument

### Removed

- Old `reference/commands` directory (migrated to `reference/skills`)
