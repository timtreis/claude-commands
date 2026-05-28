Overnight-style memory maintenance. Groom the Claude Code auto-memory system — validate, reconnect, de-duplicate, and prune the stored memories so future sessions start from a cleaner, more trustworthy brain. A lightweight take on Garry Tan's GBrain "dream cycle": no database, no embeddings, no cron — just the agent grooming its own markdown memory on demand.

## Steps

1. **Locate the brain**: Find the auto-memory directory for the current project (typically `~/.claude/projects/<project-slug>/memory/`, with `MEMORY.md` as the always-loaded index). If it doesn't exist or contains no memory files, print "nothing to groom" and stop.

2. **Inventory**: List every `*.md` memory file (excluding `MEMORY.md`). For each, read the frontmatter (`name`, `description`, `metadata.type`) and body. Build an internal table: file, name, type, one-line description, and the outbound `[[links]]` it contains.

3. **Lint (auto-fix only safe issues)**: For each memory verify —
   - frontmatter is present and parseable;
   - `name` is kebab-case and matches the filename slug;
   - `metadata.type` is one of `user` / `feedback` / `project` / `reference`;
   - `description` is a specific one-liner (not empty, not generic).
   Fix trivially-safe problems in place (e.g. normalize `name` to match the filename). Anything ambiguous — flag it, do not guess.

4. **Link integrity**: For every `[[name]]` reference, confirm a memory with that `name:` exists.
   - Dangling links are allowed by design (they mark a memory worth writing later) — **report them, do not delete**.
   - Where two memories clearly relate but aren't linked, propose adding a `[[link]]` (the user can accept).

5. **Staleness & contradiction check**: A memory that names a file path, function, flag, or config value is a claim that was true when written. Verify load-bearing claims against the *current* project: file paths via an existence check, symbols/flags via grep.
   - Flag memories that are (a) contradicted by current code, (b) reference things that no longer exist, or (c) conflict with another memory.
   - Trust current code over memory. **Do not auto-delete or auto-rewrite** — present the change with its evidence.

6. **Consolidate (dedup / merge)**: Identify memories covering the same concern (same type + overlapping topic). For each cluster, propose a single merged memory: combined body, union of `[[links]]`, the clearest `name` kept, plus a one-line rationale and the list of files that would be removed. **Propose only.**

7. **Index hygiene (orphans / purge)**: Reconcile `MEMORY.md` against the files on disk —
   - add a one-line index entry for any memory file missing from the index;
   - remove orphan index lines that point to deleted files;
   - keep each entry to one line, under ~150 chars, in the existing `- [Title](file.md) — hook` format;
   - the index must stay under 200 lines (the load truncation cap). If it's over, that is itself a consolidation signal — flag it.

8. **Report & apply**: Print the Dream Report (format below). Apply the safe auto-fixes from steps 3, 4 (reporting only), and 7. For everything destructive — merges, deletions, content rewrites — present a numbered proposal list and ask before acting.

## Rules

- Operate **only** on the auto-memory directory. Never touch source code, git, project files, or anything outside `memory/`.
- Auto-apply **only** non-destructive hygiene: frontmatter normalization, `MEMORY.md` index sync, dangling-link reporting, link suggestions.
- Merging, deleting, or rewriting memory **content** is propose-only — show the evidence, get explicit confirmation. Memory is an audit trail; when in doubt, keep it.
- Trust current code over memory. If a memory conflicts with what you can verify right now, the *memory* is the stale one — flag it for update or removal, never act on its stale claim.
- Stay lightweight. This is markdown grooming, not a knowledge graph. No databases, embeddings, external services, or new tooling.
- Be idempotent: a second run with no new memories and no accepted proposals should report "clean" and change nothing.

## Report format

```
## Dream Report — <date>
**Memories:** <N> files across <types>
**Auto-fixed:** <lint/index fixes applied, or "none">
**Dangling links:** <[[x]] with no target, or "none">
**Stale / contradicted (proposed):** <list with evidence, or "none">
**Merge candidates (proposed):** <groups, or "none">
**Index:** <K> lines (under/over the 200-line cap)
**Awaiting your OK:** <numbered list of destructive proposals, or "nothing">
```
