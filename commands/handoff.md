End-of-session handoff. Ensure all session work is documented in project-specific files before wrapping up.

## Steps

1. **Summarize the session**: Review the conversation and identify all significant work done — bugs fixed, features added, decisions made, discoveries, open questions, and blockers encountered.

2. **Update `tasks/todo.md`**: Read the current `tasks/todo.md` (create it if it doesn't exist). For each piece of session work:
   - If it's a completed fix/feature: ensure there's a checked `[x]` item describing what was done and which files were touched.
   - If it's an open issue, unfinished work, or a follow-up needed: ensure there's an unchecked `[ ]` item with enough context for a future session to pick it up.
   - If it was already tracked and completed: mark it done.
   - Do NOT remove or rewrite existing entries — only add or update.

3. **Update `tasks/lessons.md`**: If any corrections, surprises, or non-obvious patterns were discovered during the session, append them to `tasks/lessons.md` (create if needed). Each lesson should be a short rule with a "Why" line. Skip if nothing new was learned.

4. **Capture memories (auto-memory)**: This is the session's *capture* pass — the companion `/dream` command is the later *maintenance* pass (dedup, consolidate, prune). Because `/dream` consolidates afterward, **capture liberally here and let `/dream` clean up** — when in doubt, write it. Walk all four memory types and write one file per distinct fact worth carrying to a *future* session:
   - **feedback** — did the user correct an approach (*"no, don't…"*, *"stop…"*) OR confirm a non-obvious one worked (*"yes, exactly"*, accepted an unusual choice without pushback)? Capture **both** — confirmations are quiet and easy to miss but keep you from drifting away from validated approaches. Body: the rule, then a **Why:** line and a **How to apply:** line.
   - **user** — anything new about the user's role, expertise, preferences, or workflow that should shape how you collaborate next time.
   - **project** — ongoing-work context, decisions, motivations, or deadlines NOT derivable from code/git (convert relative dates to absolute, e.g. "Thursday" → "2026-03-05").
   - **reference** — external systems/resources and their purpose (dashboards, trackers, channels).

   **Filter:** skip anything derivable from current code, git history, or CLAUDE.md, and anything ephemeral to this session. Keep only the surprising / non-obvious / cross-session-useful.

   **Format each memory correctly so `/dream` has nothing to repair:** frontmatter (`name` in kebab-case matching the filename slug, `metadata.type`, and a *specific* one-line `description`), body linking related memories with `[[name]]`, AND add the matching one-line entry to `MEMORY.md`. Before creating, check existing memories — **update in place** if one already covers the concern rather than duplicating.

   **Routing vs step 3:** a technical gotcha about *this codebase* → `tasks/lessons.md`; a durable fact about the *user, project context, or how to behave* → auto-memory. If it's genuinely both, prefer the single best home.

5. **Print a handoff summary**: Output a short (5-10 line) summary formatted as:
   ```
   ## Session Handoff
   **Done:** <bullet list of completed items>
   **Open:** <bullet list of remaining/new todos>
   **Lessons:** <any new lessons added, or "none">
   **Memories:** <new/updated memory files by name, or "none">
   **Next suggested focus:** <what to pick up next session>
   ```

## Rules
- Be thorough but concise — write for a future you with no conversation context.
- Only write to `tasks/` directory and auto-memory. Do not modify source code.
- Create `tasks/` directory if it doesn't exist.
- Keep `tasks/todo.md` organized by sections/themes, not chronologically.
- Stage nothing and commit nothing — this is documentation only.
- Capture/groom loop: `/handoff` captures memories liberally; `/dream` consolidates, de-dupes, and prunes them. The pairing only works if `/dream` is run periodically — liberal capture without grooming will eventually push `MEMORY.md` past its ~200-line load cap.
- Always use `gh` (GitHub CLI) for all GitHub interactions. Do not use raw API calls, curl, or web scraping.
