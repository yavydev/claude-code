---
description: Set up Yavy documentation search - installs CLI, authenticates, and configures project skills for Claude Code
allowed-tools: ['Bash', 'AskUserQuestion']
---

You are setting up Yavy for this user. Yavy indexes documentation and makes it searchable for AI tools.

## Important Rules

- **Never create skill files yourself.** Always delegate to `yavy init` CLI.
- **Ask before every action.** Never install or run anything without the user's go-ahead.
- **Be concise.** Short sentences. Natural language.
- **Use AskUserQuestion** for choices (project selection, confirmations).

## Flow

### Step 1: Ensure CLI Installed

```bash
yavy --version 2>/dev/null
```

- **Installed:** "Yavy CLI v{version} detected."
- **Not installed:** Ask: "The Yavy CLI isn't installed. Can I run `npm install -g @yavydev/cli`?" Wait for confirmation, then install.

### Step 2: Ensure Authenticated

```bash
yavy projects --json 2>&1
```

- **Returns JSON array:** Authenticated. Continue.
- **Fails/empty:** Tell the user: "You need to log in first. Please type `! yavy login` in the prompt to authenticate in your browser." Wait for them to confirm they've logged in, then retry `yavy projects --json`.

### Step 3: Select Projects

Parse the JSON from `yavy projects --json`. Filter to projects where `has_indexed_content` is true.

- **No indexed projects:** "No indexed projects found. Add documentation sources at https://yavy.dev, then run `/yavy:init` again."
- **Has projects:** Present them via **AskUserQuestion** with options:
  - "All {N} projects" as first option
  - Each project as `{org.slug}/{name} ({pages_count} pages)`
  - Let user select

### Step 4: Run yavy init

Build the command from user selections. Use the plain `slug` field from the JSON (e.g., `knowledge-base`, NOT `org/slug`).

```bash
yavy init --tool claude-code --projects {comma-separated-slugs} 2>&1
```

- **Succeeds:** Report the summary from CLI output.
- **Fails:** Show the error and suggest retrying or checking `yavy login`.

### Step 5: Summary

> "All done! Yavy skill configured for {N} project(s). Run `/yavy:init` again to refresh or add projects."

## Gotchas

| Issue | Fix |
|---|---|
| CLI not found after install | Check PATH, try `npx yavy` instead |
| Auth expired | User runs `! yavy login` |
| `yavy init` fails | Show stderr, suggest `yavy login` or retry |
| No indexed projects | Direct user to https://yavy.dev |
