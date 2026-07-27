# Project Detection

Reference for Step 1 (analyze) and Step 2 (fill gaps) of the setup skill.

## What to read

Read whichever of these exist, and infer the stack from them:

| File | Tells you |
|---|---|
| `package.json` | Node/TypeScript; scripts block usually has every command you need |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `pyproject.toml`, `requirements.txt` | Python; check for poetry/uv/pdm |
| `Makefile` | Often the real entry point — check for `test`, `lint`, `build` targets |
| `.git/config` | Remote host hints at the issue tracker (github.com, gitlab.com) |

Also check whether a `thoughts/` directory already exists.

## What to extract

- Primary language and framework
- Package manager and available scripts
- Test commands — unit, integration, e2e (they're often distinct)
- Lint, format, build, and type-check commands
- Database tooling and its migration workflow (Prisma, SQLAlchemy/Alembic, Django ORM, Diesel, Drizzle)
- Directory structure conventions
- Issue tracker

## Detecting the issue tracker

Check for CLIs (`which linear`, `which gh`, `which glab`), a `thoughts/tickets/` or `thoughts/*/tickets/` directory, and the git remote host. Note both what the project uses *and* whether the CLI is actually installed — the generated skills depend on it.

Install hints if a CLI is missing:
- Linear: `npm install -g @linear/cli`
- GitHub: https://cli.github.com
- GitLab: https://gitlab.com/gitlab-org/cli

## Presenting findings

Show what you found as a compact list — language, framework, package manager, test/lint/format/build/typecheck commands, database, issue tracking, and whether `thoughts/` exists. Mark anything you couldn't determine as needing input rather than guessing.

## Filling gaps

Ask for everything you couldn't detect **in as few turns as possible** — every extra turn costs the user more than answering a batched prompt once.

- Command and path gaps are free-text and don't fit `AskUserQuestion`'s multiple-choice shape. Present them as a single numbered block the user fills in one pass, with a realistic example per line and an explicit opt-out (`none` / `skip`).
- The issue-tracker choice *does* fit `AskUserQuestion` (Linear / GitHub / GitLab / local files / none). Use it there, and batch any follow-up into the same call — it takes up to 4 questions.
- Only split into a second round when a later question genuinely depends on an earlier answer.

For a detected database with an unclear migration workflow, ask for two things: how schema changes are applied during development, and how formal migrations are created.

## Confirming

Present the complete resolved configuration and get a yes before generating anything. This is the last cheap moment to correct a wrong assumption.

## Detecting an existing installation

- `.claude/skills/research-codebase/SKILL.md` exists → **v2+ upgrade**, see `upgrade.md`
- `.claude/commands/research-codebase.md` exists → **v1→v2 migration**, see `upgrade.md`
- Neither → **fresh install**, continue with Step 2

## When detection fails

If you can't determine the project type at all, ask directly: what language/framework, how tests run, how linting and formatting work. If there's no test command, offer to skip test-related sections rather than inventing one.
