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

### Step 0: Check for `yavy init` (Preferred Path)

Before starting the manual flow, check if the Yavy CLI supports `init`:

```bash
yavy --version 2>/dev/null
```

- **If installed (v0.2.0+):** Delegate to the CLI's built-in init command:

  > "Yavy CLI detected! Running the guided setup..."

  ```bash
  yavy init --tool claude-code
  ```

  If `yavy init` succeeds, skip the rest of this flow entirely. If it fails, fall back to the manual steps below.

- **If not installed or older version:** Continue with the manual flow below.

### Step 1: Welcome & CLI Check

Start with a brief, warm greeting:

> "Hey! Let's get Yavy set up so you can search your indexed docs from Claude Code."

Ask the user:

> "I need to install the Yavy CLI first. Mind if I run `npm install -g @yavydev/cli`?"

Wait for confirmation, then install. If install fails, suggest alternatives (npx, local install).

After installing, try the preferred path again:

```bash
yavy init --tool claude-code
```

If it works, skip to the summary. Otherwise continue with the manual flow.

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

If the user chose **CLI Search**, create a **single** skill at `.claude/skills/yavy/SKILL.md` that covers all selected projects.

**Build the skill dynamically** from the selected projects' metadata (name, slug, description, pages_count from the `yavy projects --json` output):

1. Build a **description** string listing the technology/domain names from the selected projects, comma-separated. Example: `"Search indexed documentation for Inertia.js v2, Buckaroo payments API, oh-my-claudecode. Use when user needs reference info, asks 'how does X work', or needs to look up APIs, features, or concepts from these projects."`

2. Build a **project table** with name, slug, and page count.

3. Write this template to `.claude/skills/yavy/SKILL.md`:

```markdown
---
name: yavy
description: "{generated description with project names for triggering}"
allowed-tools: ['Bash']
---

# Yavy Documentation Search

Search indexed documentation via the Yavy CLI.

## Commands

| Command | Purpose |
|---------|---------|
| `yavy search "query"` | Search all projects |
| `yavy search "query" --project {slug}` | Search one project |
| `yavy search "query" --json` | JSON output for parsing |

## Indexed Projects

| Project | Slug | Pages |
|---------|------|-------|
| {name} | `{org}/{project}` | {count} |
| ... | ... | ... |

## When to Use --project

- User mentions a specific technology from the table above
- Query clearly scopes to one domain
- Omit when topic could span multiple projects

## Gotchas

| Issue | Fix |
|---|---|
| No results | Broaden query or remove --project |
| CLI not installed | Run `npm install -g @yavydev/cli` then `yavy login` |
| Stale project list | Re-run `/yavy:init` to refresh |
```

Replace all `{...}` placeholders with actual values. Keep the description under 500 characters.

After creating the skill, confirm:

> "Created Yavy search skill → `.claude/skills/yavy/SKILL.md` (covers {N} projects)"

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
> "The yavy skill will search your docs on-demand when you're working with related code. Your docs stay up-to-date automatically."
> "Run `/yavy:init` again anytime to add more projects or refresh the project list."

**For Skill Archive:**
> "These skills will automatically activate when you're working with related code. No extra commands needed."
> "Run `/yavy:init` again to regenerate after doc updates or to add new projects."
