# Yavy Claude Code Plugins

A collection of Claude Code plugins for searching and managing your indexed documentation on [Yavy](https://yavy.dev).

## Plugins

### yavy-docs

Interactive setup that installs the CLI, authenticates with your Yavy account, and configures documentation search for your selected projects.

**Features:**

- Installs the [Yavy CLI](https://github.com/yavydev/cli) if needed
- Authenticates via browser-based OAuth
- Lets you select which projects to set up
- Offers CLI Search (recommended) or Skill Archive approach
- Saves skill files to `.claude/skills/` for automatic activation

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add yavydev/claude-code
/plugin install yavy-docs@yavy
```

## Usage

Once installed, run the init command inside Claude Code:

```
/yavy:init
```

This walks you through the complete setup: CLI installation, authentication, project selection, and documentation search configuration.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- A [Yavy](https://yavy.dev) account with indexed documentation
- Node.js >= 20

## Related

- [Yavy CLI](https://github.com/yavydev/cli) — Search and manage your AI-ready documentation
- [Yavy](https://yavy.dev) — Index documentation, search with AI

## License

[MIT](LICENSE)
