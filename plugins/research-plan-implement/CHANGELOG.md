# Changelog

All notable changes to the `research-plan-implement` plugin are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]

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
