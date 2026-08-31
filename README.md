# Claude Code Plugins

A collection of Claude Code plugins for enhanced development workflows.

## Plugins

### Research → Design → Plan → Implement → Review

A comprehensive workflow plugin for managing AI agent context windows through intentional compaction.

**Key Features:**
- 🔬 **Research** - Explore the codebase without polluting the main context
- 🎨 **Design** - Align on approach before committing to an implementation path
- 📋 **Plan** - Create vertical phases with per-phase testing
- 🔨 **Implement** - Execute phase-by-phase with testing-aware verification
- 🔍 **Review** - Open the PR and land a numbered review guide as inline stops on the diff
- 📊 **Context Management** - Keep utilization at 40-60% for optimal results

**Commands:**
- `/research-codebase` - Research how features work
- `/design-doc` - Align on approach before planning
- `/create-plan` - Create implementation plans
- `/implement-plan` - Execute plans with verification
- `/prepare-pr` - Commit, open the PR, and land a numbered review guide
- `/guide` - Contextual orientation: where am I, what's next
- `/setup` - Generate project-specific workflow

**Installation:**
```bash
/plugin add lucasnad27/claude-plugins/research-plan-implement
```

[Read the full documentation →](plugins/research-plan-implement/README.md)

## Attribution

This workflow is inspired by [HumanLayer's](https://humanlayer.dev) research on context engineering for AI-assisted development:
- **Website:** [humanlayer.dev](https://humanlayer.dev)
- **GitHub:** [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer)
- **AI Engineering Talk:** [YouTube](https://youtu.be/rmvDxxNubIg?si=WtKgAdi6MydW8u-i)

## License

MIT
