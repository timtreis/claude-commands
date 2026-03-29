Find and suggest the next GitHub issue to work on.

**Always use `gh` (GitHub CLI) for all GitHub interactions.** Do not use raw API calls, curl, or web scraping.

## Usage

- `/next-issue` — suggest any good candidate
- `/next-issue bug` — suggest a bug fix
- `/next-issue enhancement` — suggest a feature

## Steps

1. **List open issues**:
   ```
   gh issue list --limit 20 --state open --json number,title,labels,assignees,createdAt
   ```

2. **Filter for good candidates**:
   - No assignee (unclaimed)
   - Well-scoped (clear title, has description)
   - Labels like `bug`, `enhancement`, `good-first-issue`
   - If `$ARGUMENTS` is provided, filter by that label

3. **Get details on the top candidate**:
   ```
   gh issue view <number> --json body,comments
   ```

4. **Present a recommendation** with: issue number/title, what needs doing, estimated effort, and any blockers.
