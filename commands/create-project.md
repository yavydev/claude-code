---
description: Create a new Yavy documentation project - indexes a URL or GitHub repo for AI-powered search
allowed-tools: ['Bash', 'AskUserQuestion']
---

You are helping the user create a new Yavy documentation project. Walk them through the process conversationally.

## Tool Usage Rules

**Every user-facing question MUST use the AskUserQuestion tool.** You have exactly two tools:

| Tool | When to use |
|------|-------------|
| `Bash` | Run CLI commands (`yavy`, `npm`, `open`) |
| `AskUserQuestion` | ANY question, confirmation, or choice directed at the user |

**NEVER output a question as plain text.** If you need user input, call `AskUserQuestion`.

## Other Rules

- Be concise. Short sentences. Natural language.
- If the CLI is not installed or user is not authenticated, direct them to run `/yavy:init` first.

## Flow

### Step 1: Ensure CLI and Auth

Run: `yavy projects --json 2>&1`

| Result | Action |
|--------|--------|
| Valid JSON array | Authenticated → Step 2 |
| Exit code 127 | Output: "Yavy CLI not installed. Run `/yavy:init` to set up." **Stop.** |
| Auth error | Output: "Not authenticated. Run `/yavy:init` to log in." **Stop.** |

### Step 2: Choose Source Type

Call `AskUserQuestion` with question "What type of documentation source?" and options ["Web Crawl (URL)", "GitHub Repository"].

### Step 3: Collect Source

**If Web Crawl:**
Call `AskUserQuestion` with question "What URL should Yavy crawl?" (free text input).

**If GitHub:**
Call `AskUserQuestion` with question "Which GitHub repository? (owner/repo format, e.g. laravel/docs)" (free text input).

### Step 4: Resolve Organization

Parse the JSON from Step 1 to extract unique organizations.

| Result | Action |
|--------|--------|
| 1 org | Auto-select it, tell the user: "Using organization: {org.name}" → Step 5 |
| Multiple orgs | Call `AskUserQuestion` with question "Which organization?" and options: each org as "{name} ({slug})" |
| 0 orgs | Output: "No organizations found. Create one at [yavy.dev](https://yavy.dev) first." **Stop.** |

### Step 5: Optional Settings

Call `AskUserQuestion` with question "Any additional settings?" and options:

- "Create with defaults" (recommended)
- "Set project name"
- "Make project private"
- "Skip initial sync"

If "Set project name" → call `AskUserQuestion` for the name.
If multiple settings needed, collect them sequentially.

### Step 6: Create Project

Build the command from collected inputs:

**Web Crawl:**
```
yavy project create --url "{url}" --org {org_slug} [--name "{name}"] [--private] [--no-sync] 2>&1
```

**GitHub:**
```
yavy project create --github "{owner/repo}" --org {org_slug} [--name "{name}"] [--private] [--no-sync] 2>&1
```

| Result | Action |
|--------|--------|
| Success (shows "Project created successfully") | Parse output → Step 7 |
| 422 validation error | Show the specific field errors, ask user to correct |
| Auth error | Suggest `! yavy login` and retry |
| Other error | Show error, suggest checking the URL/repo |

### Step 7: Summary

Output the project details from the CLI output (name, org, slug, MCP URL).

Then call `AskUserQuestion` with question "Want to set up the skill for this project now?" and options ["Yes, run yavy init", "No, I'm done"].

If yes → run `yavy init --tool claude-code --projects {slug} --scope project 2>&1` and report the result.

## Gotchas

| Issue | Fix |
|---|---|
| CLI not found | Direct to `/yavy:init` |
| Not authenticated | Direct to `/yavy:init` |
| Invalid URL | Ask user to check the URL format (must include https://) |
| Invalid GitHub repo | Ask user to check owner/repo format |
| Org not found | Verify slug matches what the API returns |
| Validation errors | Show the specific field error from the 422 response |
