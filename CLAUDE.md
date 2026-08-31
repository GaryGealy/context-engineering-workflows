# claude-plugins

Marketplace repo. Each plugin under `plugins/` versions independently. Releases go
through `.claude/skills/cut-a-release`.

## Testing a plugin before you release it

Load the working tree directly for one session — no marketplace entry, no config changes:

```bash
claude plugin disable <plugin-name> --scope user
cd /path/to/a/real/test/repo
claude --plugin-dir ~/dev/claude-plugins/plugins/<plugin-name>
```

Re-enable with `claude plugin enable <plugin-name>` when done.

`--plugin-dir` takes the **plugin** directory (the one holding `.claude-plugin/plugin.json`),
not the repo root. It loads alongside installed plugins, which is why the `disable` is not
optional: a `--plugin-dir` copy whose name matches an installed plugin is shadowed by the
installed one with no warning — you get the released version and think you're testing main.

`claude plugin validate <path> --strict` checks manifests without starting a session.

For a persistent local install instead, `claude plugin marketplace add ~/dev/claude-plugins`
works, but the marketplace name comes from `marketplace.json` and collides with the
GitHub-sourced one already registered. Remove that first.
