---
description: Set up Yavy documentation search - installs CLI, authenticates, and configures project skills for Claude Code
allowed-tools: ['Bash', 'Read', 'Write', 'Glob', 'AskUserQuestion']
---

You are setting up Yavy for this user. Yavy indexes documentation and makes it searchable for AI tools. Your job is to walk the user through the complete setup **conversationally** - like a friendly coworker, not a script.

## Important Guidelines

- **Ask before every action.** Never install, run, or write anything without the user's explicit go-ahead.
- **Be concise and human.** No walls of text. Short sentences. Natural language.
- **Handle errors gracefully.** If something fails, explain what happened simply and offer to retry or skip.
- **Use AskUserQuestion** for choices (project selection, confirmations with options).

## Flow

### Step 1: Welcome & CLI Check

Start with a brief, warm greeting:

> "Hey! Let's get Yavy set up so you can search your indexed docs from Claude Code."

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

- Question: "Which projects would you like to set up?"
- Options: Show each project as "{org}/{name} ({pages_count} pages)"
- Let user select multiple

### Step 4: Choose Approach

After project selection, ask the user which integration approach they prefer using **AskUserQuestion**:

- Question: "How would you like to access your documentation?"
- Options:
  1. **"CLI Search (Recommended)"** — description: "Creates a lightweight skill that searches your docs on-demand via `yavy search`. Always up-to-date, minimal disk usage."
  2. **"Skill Archive"** — description: "Downloads all documentation as local files. Works offline, but needs re-generating when docs change."

### Step 5a: CLI Search Approach

If the user chose **CLI Search**, create a skill file for each selected project.

For each project, write this SKILL.md to `.claude/skills/{project-slug}/SKILL.md`:

```markdown
---
name: {project_name}-docs
description: "Search {project_name} documentation using Yavy CLI. Use when working with {project_name} features, APIs, or concepts."
allowed-tools: ['Bash']
---

Search {project_name} documentation via the Yavy CLI.

## How to search

Run this command to find relevant documentation:

\`\`\`bash
yavy search "your question" --project {org_slug}/{project_slug} --json
\`\`\`

The `--json` flag returns structured results. Each result has:
- `title` — page title
- `url` — source URL for full context
- `content` — relevant content snippet
- `project` — the project slug

## Tips

- Use natural language queries describing what you need
- The search is semantic — it finds concepts, not just keywords
- Follow the `url` field if you need the complete page content
```

Replace `{project_name}`, `{org_slug}`, and `{project_slug}` with the actual values from the selected project.

After creating each skill file, confirm:

> "Created search skill for **{project name}** → `.claude/skills/{project-slug}/SKILL.md`"

### Step 5b: Skill Archive Approach

If the user chose **Skill Archive**, generate full skill archives for each selected project.

Confirm before generating:

> "I'll download skill archives for {N} project(s). They'll be saved to `.claude/skills/`. Want me to go ahead?"

Wait for confirmation, then generate each selected project one at a time:

```bash
yavy generate {org-slug}/{project-slug} --json
```

After each one, report the result:

> "Generated skill for **{project name}** ({reference_count} reference files) → `{path}`"

If a generation fails, offer to skip and continue:

> "The skill for {project} couldn't be generated right now. Want me to skip it and continue with the rest?"

### Step 6: Summary

After all skills are set up, provide a clean summary:

> "All done! Here's what you've got:"

List each skill with its path and approach type.

Then close based on approach:

**For CLI Search:**
> "These skills will search your docs on-demand when you're working with related code. Your docs stay up-to-date automatically."
> "Run `/yavy:init` again anytime to add more projects."

**For Skill Archive:**
> "These skills will automatically activate when you're working with related code. No extra commands needed."
> "Run `/yavy:init` again to regenerate after doc updates or to add new projects."
