# Changelog

All notable changes to the `research-plan-implement` plugin are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]

### Added

- herdr phase markers: each workflow skill tags its herdr tab with an emoji
  prefix (🔬 research · 🎨 design · 📋 plan · 🔨 implement · 🔍 review) so the
  session sidebar doubles as a phase board. Backed by a copied-verbatim
  `scripts/herdr-phase.sh` that no-ops outside herdr, so it's harmless for
  projects whose author doesn't use herdr
- `/guide herdr` topic explaining the phase markers and the manual override

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
