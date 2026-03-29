Start-of-session check-in. Review handoff documents and verify what's still relevant.

## Steps

1. **Read project state**: Read `tasks/todo.md` and `tasks/lessons.md` (if they exist). Also read the auto-memory index for project context.

2. **Identify open items**: List all unchecked `[ ]` items from `tasks/todo.md`.

3. **Validate open items**: For each open todo, do a quick relevance check:
   - If it references a specific file: verify the file still exists.
   - If it describes a bug or code smell: briefly check whether the code still has the issue (e.g., grep for the pattern described).
   - Mark items as **stale** if the referenced code no longer exists or the issue appears resolved.
   - Mark items as **current** if the issue still appears present.
   - Mark items as **unverifiable** if you can't quickly confirm either way.

4. **Check recent git history**: Run `git log --oneline -20` to see if any recent commits address open todos.

5. **Print a status report**:
   ```
   ## Session Check-in
   **Current todos:**
   - <item> — status: current / stale / unverifiable

   **Recent activity** (last N commits):
   - <one-liner per relevant commit>

   **Active lessons** (from tasks/lessons.md):
   - <key lessons to keep in mind>

   **Suggested focus:** <what seems most important to tackle>
   ```

6. **Clean up stale items**: If any items are clearly stale (code was fixed, file removed, etc.), mark them as done in `tasks/todo.md` with a note like `[x] ~~original text~~ — resolved as of <date>` or remove them if they're no longer meaningful.

## Rules
- This is a read-heavy operation — minimize writes. Only update `tasks/todo.md` to mark clearly stale items.
- Do not modify source code.
- Keep the status report concise and actionable.
- If there are no open todos and no handoff docs, just say so and ask what the user wants to work on.
