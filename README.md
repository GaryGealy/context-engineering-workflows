# Context Engineering Workflows

Plugins that manage AI coding-agent context windows through intentional compaction. Runs in Claude Code and VS Code Copilot chat.

## Plugins

### Research → Plan → Implement

A comprehensive workflow plugin for managing AI agent context windows through intentional compaction.

**Key Features:**

- 🔍 **Research Phase** - Explore codebase without polluting context
- 📋 **Plan Phase** - Create detailed implementation specifications
- ⚙️ **Implement Phase** - Execute plans with phase-by-phase verification
- 📊 **Context Management** - Keep utilization at 40-60% for optimal results

**Commands:**

- `/research-codebase` - Document how features work today
- `/design` - Align on approach before planning
- `/create-plan` - Turn a design into vertical phases with per-phase testing
- `/iterate-plan` - Update an existing plan
- `/implement-plan` - Execute phase-by-phase with verification
- `/prepare-pr` - Commit, open the PR, write it as a review guide
- `/guide` - Orientation any time; `/guide <topic>` for deep dives
- `/setup` - Generate the project-specific workflow

**Installation:**

Claude Code:

```
/plugin marketplace add GaryGealy/context-engineering-workflows
/plugin add GaryGealy/context-engineering-workflows/research-plan-implement
```

VS Code (Copilot chat):

```
Cmd/Ctrl+Shift+P
type Chat: Install Plugin From Source <enter>
type GaryGealy/context-engineering-workflows <enter>
Cmd/Ctrl+Shift+P
type Preferences: Open User Settings (JSON) <enter>
add "chat.useAgentSkills": true
```

Then run `/research-plan-implement setup` in your project. Full instructions, including what teammates need (nothing), are in the [plugin README](plugins/research-plan-implement/README.md#installation).

[Read the full documentation →](plugins/research-plan-implement/README.md)

## Attribution

The `research-plan-implement` plugin was created by [Lucas Culbertson](https://github.com/lucasnad27) in [lucasnad27/claude-plugins](https://github.com/lucasnad27/claude-plugins). This repository is a fork that continues it — most notably adding VS Code Copilot chat as a supported editor in v4.2.0.

The workflow itself is inspired by [HumanLayer's](https://humanlayer.dev) research on context engineering for AI-assisted development:

- **Website:** [humanlayer.dev](https://humanlayer.dev)
- **GitHub:** [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
- **AI Engineering Talk:** [YouTube](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i)

## License

MIT
