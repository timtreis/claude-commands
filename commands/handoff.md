End-of-session handoff. Ensure all session work is documented in project-specific files before wrapping up.

## Steps

1. **Summarize the session**: Review the conversation and identify all significant work done — bugs fixed, features added, decisions made, discoveries, open questions, and blockers encountered.

2. **Update `tasks/todo.md`**: Read the current `tasks/todo.md` (create it if it doesn't exist). For each piece of session work:
   - If it's a completed fix/feature: ensure there's a checked `[x]` item describing what was done and which files were touched.
   - If it's an open issue, unfinished work, or a follow-up needed: ensure there's an unchecked `[ ]` item with enough context for a future session to pick it up.
   - If it was already tracked and completed: mark it done.
   - Do NOT remove or rewrite existing entries — only add or update.

3. **Update `tasks/lessons.md`**: If any corrections, surprises, or non-obvious patterns were discovered during the session, append them to `tasks/lessons.md` (create if needed). Each lesson should be a short rule with a "Why" line. Skip if nothing new was learned.

4. **Update auto-memory**: Check whether any user preferences, project context, or external references were discovered that should persist across conversations. If so, write/update the appropriate memory files. Skip if nothing new.

5. **Print a handoff summary**: Output a short (5-10 line) summary formatted as:
   ```
   ## Session Handoff
   **Done:** <bullet list of completed items>
   **Open:** <bullet list of remaining/new todos>
   **Lessons:** <any new lessons added, or "none">
   **Next suggested focus:** <what to pick up next session>
   ```

## Rules
- Be thorough but concise — write for a future you with no conversation context.
- Only write to `tasks/` directory and auto-memory. Do not modify source code.
- Create `tasks/` directory if it doesn't exist.
- Keep `tasks/todo.md` organized by sections/themes, not chronologically.
- Stage nothing and commit nothing — this is documentation only.
