---
name: next-issue
description: Find and suggest the next GitHub issue to work on
argument-hint: "[optional-label-filter]"
allowed-tools: Bash(gh *)
---

# Next Issue Finder

Suggest a good GitHub issue to work on next.

## Steps

1. List open issues:
   ```
   gh issue list --limit 20 --state open --json number,title,labels,assignees,createdAt
   ```

2. Filter for good candidates:
   - No assignee (unclaimed)
   - Well-scoped (clear title, has description)
   - Labels like `bug`, `enhancement`, `good-first-issue`

3. Get details on the top candidate:
   ```
   gh issue view <number> --json body,comments
   ```

4. Present a recommendation with: issue number/title, what needs doing,
   estimated effort, and any blockers.

## Optional filter

If $ARGUMENTS is provided, filter by that label:
- `/next-issue bug` — suggest a bug fix
- `/next-issue enhancement` — suggest a feature
