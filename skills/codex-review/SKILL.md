---
name: codex-review
description: Use OpenAI Codex CLI to review code changes. Runs codex in non-interactive mode to review uncommitted changes, a specific branch diff, or a target file/directory.
---

# Codex Code Review Skill

Invoke the OpenAI Codex CLI to perform a code review from the terminal.

## Prerequisites

An `OPENAI_API_KEY` environment variable must be set (Codex reads it automatically).

## Workflow

### Step 1: Verify Codex is available

Run `which codex` to check if it's globally installed. If not, fall back to `npx @openai/codex`. Test with:

```bash
which codex 2>/dev/null && echo "GLOBAL" || (npx @openai/codex --version 2>/dev/null && echo "NPX") || echo "NOT_AVAILABLE"
```

- If **GLOBAL**: use `codex` as the command prefix.
- If **NPX**: use `npx @openai/codex` as the command prefix.
- If **NOT_AVAILABLE**: tell the user to install with `npm install -g @openai/codex` and stop.

### Step 2: Determine review scope

Ask the user what to review (if not already specified via skill arguments):

1. **Uncommitted changes** (default) - reviews staged, unstaged, and untracked files
2. **Branch diff** - reviews changes compared to a base branch (e.g., `main`)
3. **Specific path** - reviews a file or directory using a custom prompt

### Step 3: Build and run the codex command

Use the Bash tool to invoke codex. The CLI has two relevant subcommands:

- **`review`** - dedicated code review subcommand (non-interactive)
- **`exec`** - general non-interactive execution with a prompt

Use whichever command prefix was determined in Step 1 (`codex` or `npx @openai/codex`). Examples below use `CODEX` as placeholder.

**For uncommitted changes (default):**

```bash
CODEX review --uncommitted
```

**For branch diff:**

```bash
CODEX review --base {branch}
```

Replace `{branch}` with the actual base branch name (e.g., `main`).

**For a specific commit:**

```bash
CODEX review --commit {sha}
```

**For a specific path or custom review instructions:**

Use `exec` with a prompt and `--cd` to scope the working directory:

```bash
CODEX exec "Review the code in this directory. Identify bugs, security issues, and maintainability concerns. Tag each finding [P0]-[P3] with file/line citations." --cd {target_path}
```

Or pass custom instructions to `review`:

```bash
CODEX review "Focus on error handling and edge cases in the authentication module."
```

**Important flags:**
- `--full-auto` - runs with workspace-write sandbox and minimal approval prompts (add if the user wants unattended execution)
- `-s read-only` - sandbox mode, safe for review (read-only access)
- `-m {model}` - override the model (e.g., `-m o3`, `-m o4-mini`)

### Step 4: Present the results

Display the Codex output to the user. If the output contains JSON findings, format them as a readable summary:

- Group findings by priority (P0 first)
- Highlight file paths and line numbers
- Include the overall correctness verdict

### Notes

- `codex review` is a dedicated non-interactive subcommand that reads the codebase, runs the review, and streams findings before exiting.
- `codex exec` is the general non-interactive mode for arbitrary prompts.
- The review prompt is modeled after Codex's built-in `/review` slash command, which produces prioritized findings tagged P0-P3.
- Use a generous timeout (5+ minutes) as code review over large diffs can take time.
