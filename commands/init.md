---
description: Set up Yavy documentation search - installs CLI, authenticates, and configures project skills for Claude Code
allowed-tools: ['Bash', 'AskUserQuestion']
---

You are setting up Yavy for this user. Your job is to walk them through the complete setup conversationally.

## Important Rules

- **Never create skill files yourself.** Always delegate to `yavy init` CLI.
- **Ask before every action.** Never install or run anything without the user's go-ahead.
- **Be concise.** Short sentences. Natural language.
- **Use AskUserQuestion** for choices (project selection, confirmations).

## Flow

### Step 0: Welcome

Start with a brief explanation of what this plugin does:

> **Yavy** lets you index any documentation (API docs, framework guides, internal wikis) and search it directly from Claude Code.
>
> How it works:
> 1. You add documentation sources at [yavy.dev](https://yavy.dev) - Yavy crawls and indexes them
> 2. This plugin connects Claude Code to your indexed docs via the Yavy CLI
> 3. Claude can then search your docs on-demand while helping you code
>
> Let's get you set up!

### Step 1: Ensure CLI Installed

```bash
yavy --version 2>/dev/null
```

- **Installed:** "Yavy CLI v{version} detected."
- **Not installed:** Ask via **AskUserQuestion**: "The Yavy CLI isn't installed yet. Can I install it globally via npm?" with options "Yes, install it" / "No, skip". If yes, run `npm install -g @yavydev/cli`.

### Step 2: Ensure Authenticated

```bash
yavy projects --json 2>&1
```

- **Returns JSON array:** Authenticated. Continue to Step 3.
- **Fails/error:** Ask via **AskUserQuestion**:
  - Question: "Do you have a Yavy account?"
  - Options: "Yes, I need to log in" / "No, I'm new to Yavy"

  **If they have an account:**
  - Run `yavy login 2>&1` to open the browser auth flow.
  - After it completes, retry `yavy projects --json`. If still failing, suggest trying again.

  **If they're new:** Show this getting started guide:

  > **What is Yavy?**
  >
  > Yavy indexes content from any URL and makes it searchable from your AI coding tools. You can index API docs, framework guides, wikis, knowledge bases - anything with a URL.
  >
  > **How to get started:**
  >
  > 1. Go to [yavy.dev](https://yavy.dev) and create a free account
  > 2. Create an organization (this groups your projects)
  > 3. Add a project - paste any URL and Yavy will crawl and index it
  > 4. Wait for indexing to complete (usually a few minutes)
  > 5. Come back here and run `/yavy:init` again

  Then ask via **AskUserQuestion**:
  - Question: "Want me to open the registration page?"
  - Options: "Yes, open yavy.dev" / "No, I'll do it later"

  If yes, run `open https://yavy.dev/register 2>/dev/null || xdg-open https://yavy.dev/register 2>/dev/null`.

  Stop here. Do NOT continue to Step 3.

### Step 3: Select Projects

Parse the JSON from `yavy projects --json`. Filter to projects where `has_indexed_content` is true.

- **No indexed projects:** Explain:

  > Your account has no indexed projects yet. Add a project at [yavy.dev](https://yavy.dev) - paste a URL and Yavy will crawl and index it. Once done, run `/yavy:init` again.

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

> All done! Yavy skill configured for {N} project(s).
>
> Claude can now search your indexed docs while helping you code. The skill activates automatically when your questions match the indexed content.
>
> Run `/yavy:init` again anytime to add more projects or refresh.

## Gotchas

| Issue | Fix |
|---|---|
| CLI not found after install | Check PATH, try `npx yavy` instead |
| Auth expired | User runs `! yavy login` |
| `yavy init` fails | Show stderr, suggest `yavy login` or retry |
| No indexed projects | Direct user to https://yavy.dev to add docs |
| User has no account | Direct to https://yavy.dev to sign up |
