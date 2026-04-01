# Upstream Sources & Deviations

This document tracks commands ported from other tools and notes any deviations from the original.

## `adversarial-review.md`

**Source**: [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) (retrieved 2026-04-01)

**Original files**:
- `plugins/codex/prompts/adversarial-review.md` — system prompt with XML-style sections
- `plugins/codex/schemas/review-output.schema.json` — structured JSON output schema
- `plugins/codex/commands/adversarial-review.md` — command definition with frontmatter
- `plugins/codex/prompts/stop-review-gate.md` — session-end review gate prompt
- `plugins/codex/scripts/lib/git.mjs` — git context assembly logic

**Deviations**:

| Area | Original (Codex) | Our port | Reason |
|------|-------------------|----------|--------|
| Runtime | Runs via `codex-companion.mjs` calling OpenAI's app server API (GPT-5.3-Codex) | Runs inline as a Claude Code slash command | No dependency on OpenAI infrastructure |
| Output format | Structured JSON matching a JSON Schema, rendered to markdown by the companion script | Markdown output directly (same structure) | Claude Code commands output text, not JSON; the structured format is preserved in the markdown template |
| Template variables | `{{TARGET_LABEL}}`, `{{USER_FOCUS}}`, `{{REVIEW_INPUT}}` injected by companion script | Context gathered in Phase 0-1, prompt sections applied directly in Phase 2 | No template engine; the command instructs Claude to gather context then apply the review prompt |
| Execution modes | `--wait` (foreground), `--background` (Claude background task), auto-detection with `AskUserQuestion` | Always foreground | Simplicity; background mode can be added later |
| Stop-review gate | Separate `stop-review-gate.md` prompt that runs as a hook on session end, returning ALLOW/BLOCK | Not ported | Requires hook infrastructure; can be added as a separate command later |
| Frontmatter | YAML frontmatter with `disable-model-invocation`, `allowed-tools`, `argument-hint` | No frontmatter (matches repo convention) | This repo's commands don't use frontmatter |
| File size limit | Untracked files capped at 24KB in git.mjs | Same 24KB limit noted in instructions | Preserved |
| Scope flags | `--scope auto\|working-tree\|branch` | Inferred from presence/absence of `--base` | Simpler interface, same behavior |

**Not ported** (potential future additions):
- `stop-review-gate.md` — session-end review gate (ALLOW/BLOCK verdict on previous turn's code changes)
- JSON Schema validation of output
- `--background` execution mode
- `--scope` flag (currently inferred from `--base`)
