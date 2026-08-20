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
- `/research-codebase` - Research how features work
- `/create-plan` - Create implementation plans
- `/iterate-plan` - Update existing plans
- `/implement-plan` - Execute plans with verification
- `/workflow-guide` - Interactive workflow guide
- `/setup` - Generate project-specific workflow

**Installation:**
```bash
/plugin add GaryGealy/context-engineering-workflows/research-plan-implement
```

[Read the full documentation →](plugins/research-plan-implement/README.md)

## Attribution

The `research-plan-implement` plugin was created by [Lucas Culbertson](https://github.com/lucasnad27) in [lucasnad27/claude-plugins](https://github.com/lucasnad27/claude-plugins). This repository is a fork that continues it — most notably adding VS Code Copilot chat as a supported editor in v4.2.0.

The workflow itself is inspired by [HumanLayer's](https://humanlayer.dev) research on context engineering for AI-assisted development:
- **Website:** [humanlayer.dev](https://humanlayer.dev)
- **GitHub:** [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
- **AI Engineering Talk:** [YouTube](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i)

## License

MIT
