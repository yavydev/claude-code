# Yavy Claude Code Plugins

A collection of Claude Code plugins for generating AI skills from your indexed documentation on [Yavy](https://yavy.dev).

## Plugins

### yavy

Interactive setup that installs the CLI, authenticates with your Yavy account, and generates skills for your selected projects.

**Features:**

- Installs the [Yavy CLI](https://github.com/yavydev/cli) if needed
- Authenticates via browser-based OAuth
- Lets you select which projects to generate skills for
- Saves skill files to `.claude/skills/` for automatic activation

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add yavydev/claude-code
/plugin install yavy@yavydev
```

## Usage

Once installed, run the init command inside Claude Code:

```
/yavy:init
```

This walks you through the complete setup: CLI installation, authentication, project selection, and skill generation.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- A [Yavy](https://yavy.dev) account with indexed documentation
- Node.js >= 18

## Related

- [Yavy CLI](https://github.com/yavydev/cli) — Generate AI skills from the command line
- [Yavy](https://yavy.dev) — Index documentation, generate AI skills

## License

[MIT](LICENSE)
