---
description: Set up Yavy documentation search - installs CLI, authenticates, and configures project skills for Claude Code
allowed-tools: ['Bash', 'AskUserQuestion']
---

You are setting up Yavy for this user. Walk them through the setup conversationally.

## Tool Usage Rules

**Every user-facing question MUST use the AskUserQuestion tool.** You have exactly two tools:

| Tool | When to use |
|------|-------------|
| `Bash` | Run CLI commands (`yavy`, `npm`, `open`) |
| `AskUserQuestion` | ANY question, confirmation, or choice directed at the user |

**NEVER output a question as plain text.** If you need user input, call `AskUserQuestion`. If you catch yourself about to write a sentence ending in `?` — stop and use `AskUserQuestion` instead.

## Other Rules

- Never create skill files yourself — always delegate to `yavy init` CLI
- Be concise. Short sentences. Natural language.

## Flow

### Step 0: Welcome

Output this explanation (not a question, so plain text is fine):

> **Yavy** indexes content from any URL and makes it searchable from your AI coding tools.
>
> How it works:
> 1. Add any knowledge source at [yavy.dev](https://yavy.dev) — Yavy crawls and indexes it
> 2. This plugin connects Claude Code to your indexed content via the Yavy CLI
> 3. Claude searches your content on-demand while helping you code

Then proceed to Step 1.

### Step 1: Ensure CLI Installed

Run: `yavy --version 2>/dev/null`

| Result | Action |
|--------|--------|
| Version printed | Say "Yavy CLI v{version} detected." → Step 2 |
| Exit code 127 | Call `AskUserQuestion` with question "The Yavy CLI isn't installed yet. Can I install it globally via npm?" and options ["Yes, install it", "No, skip"]. If yes → run `npm install -g @yavydev/cli 2>&1` → Step 2 |

### Step 2: Ensure Authenticated

Run: `yavy projects --json 2>&1`

| Result | Action |
|--------|--------|
| Valid JSON array | Authenticated → Step 3 |
| Error/failure | Call `AskUserQuestion` with question "Do you have a Yavy account?" and options ["Yes, I need to log in", "No, I'm new to Yavy"] |

**If "Yes, I need to log in":**
- Run `yavy login 2>&1` to open the browser auth flow
- After it completes, retry `yavy projects --json`. If still failing, suggest trying again.

**If "No, I'm new to Yavy":**

Output this guide:

> **What is Yavy?**
>
> Yavy indexes content from any URL and makes it searchable from your AI coding tools. You can index API docs, framework guides, wikis, knowledge bases — anything with a URL.
>
> **How to get started:**
>
> 1. Go to [yavy.dev](https://yavy.dev) and create a free account
> 2. Create an organization (this groups your projects)
> 3. Add a project — paste any URL and Yavy will crawl and index it
> 4. Wait for indexing to complete (usually a few minutes)
> 5. Come back here and run `/yavy:init` again

Then call `AskUserQuestion` with question "Want me to open the registration page?" and options ["Yes, open yavy.dev", "No, I'll do it later"]. If yes → run `open https://yavy.dev/register 2>/dev/null || xdg-open https://yavy.dev/register 2>/dev/null`. **Stop here.** Do NOT continue to Step 3.

### Step 3: Select Projects

Parse the JSON from Step 2. Filter to projects where `has_indexed_content` is true.

| Result | Action |
|--------|--------|
| No indexed projects | Output: "Your account has no indexed projects yet. Add a project at yavy.dev — paste a URL and Yavy will crawl and index it. Once done, run `/yavy:init` again." **Stop here.** |
| Has projects | Call `AskUserQuestion` with question "Which projects do you want to set up?" and options: first option "All {N} projects", then each project as "{org.slug}/{name} ({pages_count} pages)" |

### Step 4: Choose Scope

Call `AskUserQuestion` with question "Where should the skill be installed?" and options ["This project only", "All my projects (global)"]. Map the answer:

| Answer | `--scope` value |
|--------|-----------------|
| This project only | `project` |
| All my projects (global) | `user` |

### Step 5: Run yavy init

Build the command from user selections. Use the plain `slug` field from the JSON (e.g., `knowledge-base`, NOT `org/slug`).

Run: `yavy init --tool claude-code --projects {comma-separated-slugs} --scope {scope} 2>&1`

| Result | Action |
|--------|--------|
| Success | Report summary from CLI output → Step 6 |
| Failure | Show the error, suggest retrying or running `! yavy login` |

### Step 6: Summary

> All done! Yavy skill configured for {N} project(s).
>
> Claude can now search your indexed content while helping you code. The skill activates automatically when your questions match the indexed content.
>
> Run `/yavy:init` again anytime to add more projects or refresh.

## Gotchas

| Issue | Fix |
|---|---|
| CLI not found after install | Check PATH, try `npx yavy` instead |
| Auth expired | User runs `! yavy login` |
| `yavy init` fails | Show stderr, suggest `yavy login` or retry |
| No indexed projects | Direct user to https://yavy.dev to add content |
| User has no account | Direct to https://yavy.dev to sign up |
