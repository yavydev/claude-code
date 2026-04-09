# Yavy - Claude Code Plugin

Index any knowledge source at [yavy.dev](https://yavy.dev) and search it from Claude Code.

## What is Yavy?

[Yavy](https://yavy.dev) crawls and indexes content from any URL - API docs, framework guides, wikis, knowledge bases, internal docs. Once indexed, you can search it from your AI coding tools (Claude Code, Cursor, VS Code, and more).

This plugin connects Claude Code to your Yavy account so Claude can search your indexed content on-demand while helping you code.

## Quick Start

### 1. Install the plugin

```
/plugin marketplace add yavydev/claude-code
/plugin install yavy-docs@yavy
/reload-plugins
```

### 2. Run the setup

```
/yavy:init
```

This walks you through CLI installation, authentication, project selection, and skill configuration. If you're new to Yavy, it'll guide you through creating an account too.

### 3. Create a project (optional)

```
/yavy:create-project
```

Create a new documentation project directly from Claude Code. Supports web crawl (any URL) and GitHub repositories.

## New to Yavy?

1. Go to [yavy.dev](https://yavy.dev) and create a free account
2. Create an organization (this groups your projects)
3. Add a project - paste any URL and Yavy will crawl and index it
4. Wait for indexing to complete (usually a few minutes)
5. Come back and run `/yavy:init`

## How It Works

1. `/yavy:init` installs the [Yavy CLI](https://github.com/yavydev/cli) and authenticates with your account
2. You select which indexed projects to connect
3. The CLI creates a skill file in `.claude/skills/` that Claude Code picks up automatically
4. When you ask questions that match your indexed content, Claude searches them via `yavy search`

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Node.js >= 20
- A [Yavy](https://yavy.dev) account (free)

## Related

- [Yavy CLI](https://github.com/yavydev/cli) - search and manage your indexed content
- [Yavy](https://yavy.dev) - index any knowledge source, search with AI

## License

[MIT](LICENSE)
