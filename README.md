# Yavy – Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that generates AI skills from your indexed documentation on [Yavy](https://yavy.dev).

## What It Does

Yavy indexes your documentation sources (websites, GitHub repos, Confluence, Notion) and generates Claude Code skill files from them. Skills give Claude deep knowledge of your specific tools, frameworks, and APIs — so it can write better code with fewer hallucinations.

This plugin provides:

- **`/yavy:init`** — Interactive setup: installs the CLI, authenticates, and generates skills for your selected projects.

## Installation

Install via Claude Code:

```bash
claude install-plugin yavydev/claude-code
```

## Usage

After installing, run the init command inside Claude Code:

```
/yavy:init
```

This will walk you through:

1. Installing the [Yavy CLI](https://github.com/yavydev/cli) (if needed)
2. Authenticating with your Yavy account
3. Selecting which projects to generate skills for
4. Saving skill files to `.claude/skills/`

Generated skills activate automatically when Claude Code detects relevant context.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- A [Yavy](https://yavy.dev) account with indexed documentation
- Node.js >= 18

## Related

- [Yavy CLI](https://github.com/yavydev/cli) — The command-line tool for generating skills
- [Yavy](https://yavy.dev) — Index documentation, generate AI skills

## License

[MIT](LICENSE)
