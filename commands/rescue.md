Recover a crashed session. When a slurm node (or any session) dies before `/handoff` could run, reconstruct the lost work from the on-disk transcript, rehydrate it into the current session, and write the handoff docs retroactively. Triggers: "recover the chat", "rescue the session", "the node died", "restore the lost session".

## Background

Claude Code stores every session as a JSONL transcript at:

```
~/.claude/projects/<slug>/<session-uuid>.jsonl
```

where `<slug>` is the current working directory with every `/` replaced by `-`. This makes recovery **project-scoped for free**: derive the slug from `$PWD` and you are only ever looking at sessions for *this* project. Each JSONL line is one event (user message, assistant text, `tool_use` with file paths / bash commands, `tool_result`, timestamps).

## Steps

1. **Locate this project's transcript directory.**
   - Compute the slug: take `$PWD`, replace every `/` with `-` (the leading slash becomes a leading `-`). Example: `/ictstr01/groups/ml01/workspace/ttreis/projects/spatialdata-io-ns` → `-ictstr01-groups-ml01-workspace-ttreis-projects-spatialdata-io-ns`.
   - The directory is `~/.claude/projects/<slug>/`. If it does not exist, report that and stop.

2. **Identify the live (current) session and exclude it.**
   - The transcript being written *right now* is the only one containing this very `/rescue` invocation. Find it by grepping the candidate files for the rescue trigger text and exclude the match. Never recover the live session — that is the one you are already in.

3. **Build a recovery menu.** List the remaining `.jsonl` files newest-first by mtime. For each, parse it (use `jq` or a short python snippet — do **not** read the multi-MB raw file into context) and show a one-line preview:
   - last-modified time,
   - the first user request (what the session set out to do),
   - the last user message and last assistant action (where it was cut off),
   - rough counts: # of `Edit`/`Write` events, # of `Bash` git/test commands.
   - Present these as a numbered menu and **ask the user which session to recover.** Always show the menu — do not auto-pick. If there is only one candidate, still confirm before parsing it.

4. **Reconstruct the chosen session.** Parse the selected transcript end-to-end (via `jq`/python, streaming — never dump raw JSONL into context) and extract a structured picture:
   - The user's goals and any explicit decisions or constraints they stated.
   - Every file touched, with the paths (`Edit`/`Write`/`NotebookEdit` inputs).
   - Bash commands of consequence — especially `git` (commits, branches, stashes) and test runs, with their outcomes if captured in `tool_result`.
   - Discoveries, surprises, and open questions raised.
   - The final few exchanges, so the precise cut-off point is clear.

5. **Reconcile against disk.** The transcript shows what was *attempted*; the working tree shows what *survived* the crash. Run `git status` and `git diff` (and `git stash list`) to determine:
   - which edits actually landed on disk,
   - which were in-flight / uncommitted when the node died,
   - whether anything described in the transcript is now missing and must be redone.
   - Call out any divergence explicitly — this is the highest-value part of recovery.

6. **Write the handoff docs retroactively** (this is a rehydrate **and** document command):
   - Update `tasks/todo.md`: checked `[x]` items for work that completed and landed, unchecked `[ ]` items for anything in-flight, lost, or still open — with enough context to resume cold. Do not remove or rewrite existing entries; only add/update. Organize by theme, not chronologically.
   - Append to `tasks/lessons.md` any corrections, surprises, or non-obvious patterns the dead session discovered (short rule + a "Why" line). Skip if nothing new.
   - Update auto-memory if durable user preferences / project context / external references surfaced. Skip if nothing new.
   - Create the `tasks/` directory if it does not exist.

7. **Print a recovery summary** so work can continue in this session:
   ```
   ## Session Rescue
   **Recovered from:** <transcript filename> (<last-modified time>)
   **Goal of dead session:** <one line>
   **Landed on disk:** <bullets — committed/saved work confirmed via git>
   **In-flight / lost:** <bullets — attempted but uncommitted or missing>
   **Cut off at:** <what was happening in the final exchange>
   **Docs written:** <todo.md / lessons.md / memory updates, or "none">
   **Next step:** <the single most useful thing to resume with>
   ```

## Rules
- **Never** read a raw transcript into context — they are large. Always extract with `jq` or a short streaming python script and only surface the distilled result.
- Recovery is read-only with respect to source code: do not modify source files. Only write to `tasks/` and auto-memory.
- Trust the working tree over the transcript when they disagree — the transcript records intent, the disk records reality. Surface the conflict; don't silently assume the transcript won.
- Always show the session menu and let the user choose. Never auto-recover.
- Stage nothing and commit nothing — recovery is documentation only.
- Always use `gh` (GitHub CLI) for any GitHub interactions. Do not use raw API calls, curl, or web scraping.
