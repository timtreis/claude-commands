# claude-commands

Reusable Claude Code custom slash commands and skills.

## Commands

- **`/checkin`** — Start-of-session check-in. Reviews handoff documents, validates open todos against current code state, and prints a status report.
- **`/handoff`** — End-of-session handoff. Summarizes session work, updates todo/lessons files, and prints a handoff summary for the next session.
- **`/investigate-issue`** — Deep analysis of a GitHub issue: reproduces the bug, creates a fix plan, implements, and adds tests. Usage: `/investigate-issue 123`.
- **`/adversarial-review`** — Adversarial code review that challenges design choices, tradeoffs, and assumptions. Ported from [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc). Usage: `/adversarial-review --base main [focus area]`.
- **`/dream`** — Overnight-style memory maintenance. Grooms the Claude Code auto-memory system (lint, link integrity, staleness checks, dedup, guarded synthesis, index hygiene) so future sessions start from a cleaner brain. A lightweight take on Garry Tan's GBrain "dream cycle" — markdown only, no database or cron.
- **`/setup-diary`** — Scaffold a self-contained "working diary" into the current repo: a `CLAUDE.md` protocol that tells Claude to log every task, a `tasks.js` data file it maintains, a `diary.html` viewer (project name, auto-refresh, status filters, per-task cards), and a `diary-serve.sh` helper for viewing over a forwarded port on a remote/SSH host (e.g. a cluster). Idempotent — preserves an existing `tasks.js`.
- **`/clean-review`** — Runs the `code-review` skill at max effort, aimed at cleanliness: fewer lines, leaner comments. Takes an optional PR number, branch, or path; reviews the current diff otherwise.

## Skills

- **`next-issue`** — Find and suggest the next GitHub issue to work on. Optionally filter by label (e.g., `/next-issue bug`).

## Installation

Symlink the commands into your `~/.claude/commands/` directory:

```bash
# Clone this repo (if not already)
git clone https://github.com/timtreis/claude-commands.git ~/claude-commands

# Create symlinks for all commands
mkdir -p ~/.claude/commands
for cmd in ~/claude-commands/commands/*.md; do
  ln -sf "$cmd" ~/.claude/commands/"$(basename "$cmd")"
done

# ...and for all skills (a skill is a directory holding SKILL.md)
mkdir -p ~/.claude/skills
for skill in ~/claude-commands/skills/*/; do
  ln -sfn "${skill%/}" ~/.claude/skills/"$(basename "$skill")"
done
```

To update, just `git pull` in the repo — symlinks pick up changes automatically.
The repo is the source of truth: nothing should live only in `~/.claude`.
