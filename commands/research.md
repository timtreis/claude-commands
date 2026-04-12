Research a topic deeply and write the findings to a standalone markdown artifact in `tasks/` so the coach vault's `/research-ingest` skill can promote it into the knowledgebase later.

## When to use

Invoke when the user asks to "research X", "look up Y", "investigate Z", "dig into W", or any open-ended knowledge-gathering request that is bigger than a one-shot answer. Do NOT use for small lookups — use this when the answer deserves to be remembered.

## Contract

The output of this command is a **standalone markdown file**, not a chat reply. The chat reply is a short summary + the path to the written file. Treat the file as the deliverable.

## Steps

1. **Clarify the question** (silent unless ambiguous).
   - What exactly is being asked? Phrase it as a single researchable question.
   - What is the scope? (paper comparison, dataset metadata, method gotchas, infra decision, debugging investigation, …)
   - If genuinely ambiguous, ask ONE clarifying question before starting. Otherwise proceed.

2. **Pick an archetype**. The archetype determines the filename, title, and structural template. These match the `research-ingest` skill's taxonomy so ingest is automatic.

   | Archetype | Use when | Filename pattern |
   |-----------|----------|------------------|
   | `reference-dataset` | Documenting an external dataset, corpus, or benchmark | `<dataset>_research.md` |
   | `reference-paper` | Comparing to / summarizing published methods | `<topic>_baseline.md` |
   | `reference-ops` | Infra, tooling, container, migration decisions | `<topic>_migration.md` or `<topic>_infra.md` |
   | `investigation` | Root-cause or failure analysis with a concrete status | `<component>_<symptom>.md` |
   | `project-research` | Scoped plan, registry, or design doc for this one project | `<thing>_registry.md` or `<thing>_plan.md` |

3. **Gather the material**. Use the tools appropriate to the archetype:
   - `Read` / `Grep` / `Glob` — read actual code, configs, data files in the current repo before making any claims about "what the pipeline does". Cite file paths and line numbers.
   - `WebFetch` / `WebSearch` — for external papers, datasets, docs. Always capture DOI, URL, and access date.
   - `Bash` — for reproducing behavior, inspecting file headers, counting rows, running `--help`. Never mutate state.
   - Sub-agents — delegate parallel subqueries if the scope is broad.

   **Rule**: every factual claim in the output must be traceable to either a source citation (paper, URL, DOI) or a code location (`path/to/file.py:123`). Uncited claims get flagged as UNGROUNDED by vault hygiene later — don't ship them.

4. **Write the file** to `tasks/<archetype-pattern>.md` (create `tasks/` if missing). Use this template:

   ```markdown
   # <Sharp title — this becomes the vault note's H1>

   **Date**: YYYY-MM-DD
   **Status**: <In progress | Investigated, fix proposed | Complete | Superseded by …>
   **Question**: <the single researchable question from step 1>
   **Scope**: <one sentence bounding what this does and does not cover>

   ---

   ## Summary

   <3-6 sentence plain-language TL;DR. A reader should be able to stop here and get the answer.>

   ## Background

   <Context the reader needs. Why does this question exist? What is the system under study?>

   ## Findings

   <The substance. Use H3s for sub-findings. Tables for anything enumerable. Code blocks for exact commands, schemas, configs. Inline citations for every factual claim: `[Author Year]`, `[file:line]`, or `[URL]`.>

   ## Evidence

   <Optional: raw outputs, query results, diff snippets, tables of measured numbers. The "show your work" section.>

   ## Open questions

   <Things this research did NOT resolve. These become open-loops on ingest.>

   ## Sources

   - <Paper title> — DOI, accessed YYYY-MM-DD
   - <URL> — accessed YYYY-MM-DD
   - `<repo-path>:<line>` — local code reference
   ```

5. **Verbatim rule**. Do not summarize findings in the file just because they're long. Ingest is lossy enough — the research artifact should be the **full** record. Tables, quoted paragraphs from papers, exact error messages, dataset sizes: keep them all.

6. **Chat output**. Reply in the chat with:
   - One-line TL;DR
   - The path to the written file
   - The open questions (so the user can redirect before commit)

   Do NOT dump the file contents into the chat. The file is the artifact.

## Rules

- **One topic per file.** If the research splits into two genuinely distinct topics, write two files and link them with a `See also:` line.
- **Title from the question, not the filename.** Filenames use snake_case for pattern matching; the H1 is a proper sentence.
- **Never overwrite** an existing research file silently. If `tasks/<name>.md` exists, append a new `## Update — YYYY-MM-DD` section instead.
- **Never commit** as part of this command. Research is a draft artifact; `/handoff` handles commits.
- **Scope discipline**: if the user asks about X and you discover Y is also interesting, note Y under "Open questions" — do not silently expand the scope.
- **Citations are mandatory.** No claims without a source or a code reference. This is what makes the artifact useful to `/research-ingest`.

## Why this shape

This command is designed to hand off cleanly to the coach vault's `/research-ingest` skill. That skill:
- discovers `tasks/*.md` files matching the archetype filename patterns
- uses the H1 (not the filename) to slug the vault target path
- expects a `Date:` + `Status:` preamble to route `investigation` archetype notes
- expects `DOI:` / `Journal:` markers for `reference-paper` routing
- harvests "Open questions" into the vault's open-loops file
- treats the body as verbatim source of truth for the eventual knowledgebase note

So: writing in this shape means the research will be automatically promoted into the permanent knowledgebase the next time `/research-ingest` runs. Deviating from the shape means the work stays stuck in the repo.
