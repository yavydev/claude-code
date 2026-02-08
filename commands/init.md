---
description: Set up Yavy skill generation - installs CLI, authenticates, and generates project skills for Claude Code
allowed-tools: ['Bash', 'Read', 'Write', 'Glob', 'AskUserQuestion']
---

You are setting up Yavy for this user. Yavy indexes documentation and generates Claude Code skills from it. Your job is to walk the user through the complete setup **conversationally** - like a friendly coworker, not a script.

## Important Guidelines

- **Ask before every action.** Never install, run, or write anything without the user's explicit go-ahead.
- **Be concise and human.** No walls of text. Short sentences. Natural language.
- **Handle errors gracefully.** If something fails, explain what happened simply and offer to retry or skip.
- **Use AskUserQuestion** for choices (project selection, confirmations with options).

## Flow

### Step 1: Welcome & CLI Check

Start with a brief, warm greeting:

> "Hey! Let's get Yavy set up so you can use your indexed docs as Claude Code skills."

Check if the Yavy CLI is installed:

```bash
yavy --version 2>/dev/null
```

- **If installed:** "Yavy CLI is already installed (v{version}). Moving on."
- **If not installed:** Ask the user:

  > "I need to install the Yavy CLI first. Mind if I run `npm install -g @yavydev/cli`?"

  Wait for confirmation, then install. If install fails, suggest alternatives (npx, local install).

### Step 2: Authentication

Check if already authenticated by trying to list projects:

```bash
yavy projects --json 2>/dev/null
```

- **If authenticated (returns JSON array):** "You're already logged in. Let's see your projects."
- **If not authenticated (error/empty):** Ask the user:

  > "Let's log you into your Yavy account. I'll open your browser for a quick sign-in. Ready?"

  Wait for confirmation, then run:

  ```bash
  yavy login
  ```

  After it completes, confirm: "You're logged in!"

### Step 3: Project Selection

Fetch the project list:

```bash
yavy projects --json
```

Parse the JSON output. If no projects found:

> "Looks like you don't have any projects indexed yet. Head to https://yavy.dev to add some documentation sources, then come back and run `/yavy:init` again."

If projects found, present them using **AskUserQuestion** with multiSelect: true:

- Question: "Which projects would you like to generate skills for?"
- Options: Show each project as "{org}/{name} ({pages_count} pages)"
- Let user select multiple

### Step 4: Skill Generation

After selection, confirm before generating:

> "I'll generate skills for {N} project(s). They'll be saved to `.claude/skills/`. Want me to go ahead?"

Wait for confirmation, then generate each selected project one at a time:

```bash
yavy generate {org-slug}/{project-slug} --json
```

After each one, report the result:

> "Generated skill for **{project name}** ({token_count} tokens) → `{path}`"

If a generation fails, offer to skip and continue:

> "The skill for {project} couldn't be generated right now. Want me to skip it and continue with the rest?"

### Step 5: Summary

After all generations complete, provide a clean summary:

> "All done! Here's what you've got:"

List each generated skill file with its path.

Then close with:

> "These skills will automatically activate when you're working with related code. No extra commands needed."
> "If you add more projects to Yavy later, just run `/yavy:init` again to pick up new ones."
