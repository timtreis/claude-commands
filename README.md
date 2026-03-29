# claude-commands

Reusable Claude Code custom slash commands.

## Commands

- **`/checkin`** — Start-of-session check-in. Reviews handoff documents, validates open todos against current code state, and prints a status report.
- **`/handoff`** — End-of-session handoff. Summarizes session work, updates todo/lessons files, and prints a handoff summary for the next session.
- **`/next-issue`** — Find and suggest the next GitHub issue to work on. Optionally filter by label (e.g., `/next-issue bug`).
- **`/investigate-issue`** — Deep analysis of a GitHub issue: reproduces the bug, creates a fix plan, implements, and adds tests. Usage: `/investigate-issue 123`.

## Installation

Symlink the commands into your `~/.claude/commands/` directory:

```bash
# Clone this repo (if not already)
git clone https://github.com/timtreis/claude-commands.git ~/claude-commands

# Create symlinks for all commands
for cmd in ~/claude-commands/commands/*.md; do
  ln -sf "$cmd" ~/.claude/commands/"$(basename "$cmd")"
done
```

To update, just `git pull` in the repo — symlinks pick up changes automatically.
