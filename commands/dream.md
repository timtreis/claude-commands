Overnight-style memory maintenance. Groom the Claude Code auto-memory system — validate, reconnect, de-duplicate, and prune the stored memories so future sessions start from a cleaner, more trustworthy brain. A lightweight take on Garry Tan's GBrain "dream cycle": no database, no embeddings, no cron — just the agent grooming its own markdown memory on demand.

## Steps

1. **Locate the brain**: Find the auto-memory directory for the current project (typically `~/.claude/projects/<project-slug>/memory/`, with `MEMORY.md` as the always-loaded index). If it doesn't exist or contains no memory files, print "nothing to groom" and stop.

2. **Inventory**: List every `*.md` memory file (excluding `MEMORY.md`). For each, read the frontmatter (`name`, `description`, `metadata.type`) and body. Build an internal table: file, name, type, one-line description, and the outbound `[[links]]` it contains.

3. **Lint (auto-fix only safe issues)**: For each memory verify —
   - frontmatter is present and parseable (note `type` is usually nested under `metadata:`);
   - `name` is kebab-case and matches the filename slug;
   - `metadata.type` is one of `user` / `feedback` / `project` / `reference`;
   - the **filename prefix matches the declared type** — a `user_…` file typed `feedback` is a smell. **Flag it, do not auto-fix**: either the prefix or the type could be the intended one, and the content decides which.
   - `description` is a specific one-liner (not empty, not generic).
   Fix trivially-safe problems in place (e.g. normalize `name` to match the filename). Anything ambiguous — flag it, do not guess.

4. **Link integrity**: For every `[[name]]` reference, confirm a memory with that `name:` exists.
   - Dangling links are allowed by design (they mark a memory worth writing later) — **report them, do not delete**.
   - Where two memories clearly relate but aren't linked, propose adding a `[[link]]` (the user can accept).

5. **Staleness & contradiction check**: A memory that names a file path, function, flag, or config value is a claim that was true when written. Verify load-bearing claims against the *current* project: file paths via an existence check, symbols/flags via grep.
   - Flag memories that are (a) contradicted by current code, (b) reference things that no longer exist, or (c) conflict with another memory.
   - Trust current code over memory. **Do not auto-delete or auto-rewrite** — present the change with its evidence.

6. **Consolidate (dedup / merge)**: Identify memories covering the same concern (same type + overlapping topic). For each cluster, propose a single merged memory: combined body, union of `[[links]]`, the clearest `name` kept, plus a one-line rationale and the list of files that would be removed. **Propose only.**

7. **Synthesis candidates (propose-only, guarded)**: Look across *distinct* memories for a higher-order insight that none of them states alone — the "wake up smarter" step. Where several memories point at one root pattern, propose a single new synthesized memory.
   - Only propose when a candidate draws on **≥3 distinct primary memories**. Never synthesize from a single memory or from a near-trivial pair (that's step 6's job).
   - **A shared *topic* is not enough.** The synthesis must assert something none of the source memories states on its own, AND that isn't already captured in a higher-level doc (e.g. `CLAUDE.md`). If the cluster is merely "these are all about X," do **not** propose it — that's the spurious-pattern trap. A real candidate reveals a connection, not a category.
   - A proposed synthesis is a *new* memory carrying provenance frontmatter: `metadata.origin: dream-synthesis`, `metadata.synthesized_from: [name-a, name-b, name-c]`, `metadata.synthesized_on: <date>`. If accepted, each source memory gets `metadata.consolidated_into: <new-name>` and is kept as an audit trail.
   - **Propose only.** Present: proposed name + type, the one-line insight, the source memories, and what happens to them. Write nothing without explicit confirmation.
   - See the self-consumption guard in Rules — synthesis is the one generative step, so it is fenced hardest.

8. **Index hygiene (orphans / purge)**: Reconcile `MEMORY.md` against the files on disk —
   - add a one-line index entry for any memory file missing from the index;
   - remove orphan index lines that point to deleted files;
   - keep each entry to one line, under ~150 chars, in the existing `- [Title](file.md) — hook` format;
   - the index must stay under 200 lines (the load truncation cap). If it's over, that is itself a consolidation signal — flag it.

9. **Report & apply**: Print the Dream Report (format below). Apply the safe auto-fixes from steps 3, 4 (reporting only), and 8. For everything destructive — merges, deletions, content rewrites, and all synthesis — present a numbered proposal list and ask before acting.

## Rules

- Operate **only** on the auto-memory directory. Never touch source code, git, project files, or anything outside `memory/`.
- Auto-apply **only** non-destructive hygiene: frontmatter normalization, `MEMORY.md` index sync, dangling-link reporting, link suggestions.
- Merging, deleting, or rewriting memory **content** is propose-only — show the evidence, get explicit confirmation. Memory is an audit trail; when in doubt, keep it.
- Trust current code over memory. If a memory conflicts with what you can verify right now, the *memory* is the stale one — flag it for update or removal, never act on its stale claim.
- Stay lightweight. This is markdown grooming, not a knowledge graph. No databases, embeddings, external services, or new tooling.
- Be idempotent: a second run with no new memories and no accepted proposals should report "clean" and change nothing.

### Synthesis self-consumption guard

Synthesis is the only step that *creates* new memory, so it eats its own output unless fenced. Enforce all of these:

- **Depth-1 only.** A memory with `origin: dream-synthesis` is **never** an input to another synthesis. Derived memory cannot beget derived memory — this caps the chain at one hop so noise can't compound run over run.
- **Provenance is mandatory.** Every synthesized memory records `synthesized_from` + `origin`. If you can't record where it came from, don't write it — that's what makes future exclusion possible.
- **Skip already-covered clusters (idempotency).** Before proposing, check existing `synthesized_from` sets. If a candidate's sources are already covered by a synthesized memory, skip it. A clean brain produces zero synthesis proposals on a repeat run.
- **Exclude consolidated sources.** A memory marked `consolidated_into` is spent — never re-feed it into synthesis (otherwise the same cluster regenerates forever).
- **Cap per run.** Surface at most ~3 synthesis proposals per run, strongest first; note the rest rather than dumping them.
- **Human gate, always.** Synthesis never auto-writes and never auto-retires sources. The user is the final guard; the rules above keep the *proposals* themselves from degrading over time.

## Report format

```
## Dream Report — <date>
**Memories:** <N> files across <types>
**Auto-fixed:** <lint/index fixes applied, or "none">
**Dangling links:** <[[x]] with no target, or "none">
**Stale / contradicted (proposed):** <list with evidence, or "none">
**Merge candidates (proposed):** <groups, or "none">
**Synthesis candidates (proposed):** <new-memory ideas w/ sources, or "none">
**Index:** <K> lines (under/over the 200-line cap)
**Awaiting your OK:** <numbered list of destructive + synthesis proposals, or "nothing">
```
