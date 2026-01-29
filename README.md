# codex-review

A [Claude Code](https://claude.ai/code) plugin that invokes [OpenAI Codex CLI](https://github.com/openai/codex) to review your code changes.

Type `/codex-review` in Claude Code to get a prioritized code review powered by Codex — without leaving your terminal.

## What it does

- Runs `codex review` non-interactively from Claude Code
- Reviews uncommitted changes, branch diffs, specific commits, or custom targets
- Returns prioritized findings tagged `[P0]`–`[P3]` with file/line citations
- Auto-detects whether Codex is installed globally or falls back to `npx`

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- [OpenAI Codex CLI](https://github.com/openai/codex) installed (`npm install -g @openai/codex`) or available via `npx`
- `OPENAI_API_KEY` environment variable set

## Install

```bash
/plugin install https://github.com/shahdpinky/codex-review-skill
```

Or clone and load locally:

```bash
git clone https://github.com/shahdpinky/codex-review-skill.git
claude --plugin-dir ./codex-review-skill
```

## Usage

In Claude Code, run:

```
/codex-review
```

You'll be asked what to review:

| Scope | Codex command |
|-------|--------------|
| Uncommitted changes (default) | `codex review --uncommitted` |
| Branch diff | `codex review --base main` |
| Specific commit | `codex review --commit abc1234` |
| Custom prompt | `codex review "focus on error handling"` |
| Specific directory | `codex exec "review this code" --cd src/` |

## Example output

```
OpenAI Codex v0.92.0 (research preview)
--------
model: gpt-5.2-codex
--------

findings:
  [P1] Unchecked null return from API call
       src/api/client.py:42
       The response is used without checking for None...

  [P2] Missing timeout on HTTP request
       src/api/client.py:38
       requests.get() called without timeout parameter...

overall_correctness: "patch is correct"
```

## How it works

This plugin wraps the Codex CLI's dedicated `review` subcommand, which uses the same review prompt as Codex's built-in `/review` slash command. Findings are prioritized:

- **P0** — Drop everything. Blocking release or operations.
- **P1** — Urgent. Should be addressed in the next cycle.
- **P2** — Normal. To be fixed eventually.
- **P3** — Low. Nice to have.

## License

MIT
