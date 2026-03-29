# claude-commands

Reusable Claude Code custom slash commands.

## Commands

- **`/checkin`** — Start-of-session check-in. Reviews handoff documents, validates open todos against current code state, and prints a status report.
- **`/handoff`** — End-of-session handoff. Summarizes session work, updates todo/lessons files, and prints a handoff summary for the next session.

## Installation

Symlink the commands into your `~/.claude/commands/` directory:

```bash
# Clone this repo (if not already)
git clone https://github.com/timtreis/claude-commands.git ~/claude-commands

# Create symlinks
ln -sf ~/claude-commands/commands/checkin.md ~/.claude/commands/checkin.md
ln -sf ~/claude-commands/commands/handoff.md ~/.claude/commands/handoff.md
```

To update, just `git pull` in the repo — symlinks pick up changes automatically.
