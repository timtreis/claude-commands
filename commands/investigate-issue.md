Deep analysis of a GitHub issue — reproduces, plans, implements fix, and adds tests.

## Usage

- `/investigate-issue 123` — investigate issue #123
- `/investigate-issue https://github.com/org/repo/issues/123` — investigate from URL

Detect the repo owner/name from the current git remote. If the argument is a plain number, treat it as an issue in this repo. If it's a full URL, extract the issue number from it.

Execute the following phases **in order**. Do NOT skip phases. After each phase, print a brief status summary before moving on.

---

## Phase 0: Resolve issue reference and bootstrap environment

### 0a — Resolve the issue

```
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
```

Parse `$ARGUMENTS` into an issue number. Validate the issue exists:
```
gh issue view <number> --json title,body,state,labels,assignees,milestone
```

**Important:** Always use `--json` with `gh issue view` to avoid deprecated API errors from the default display format.

### 0b — Bootstrap the project environment

Before any code execution, ensure the project environment is ready. **Always use the repo's official tooling** — never install packages manually or create ad-hoc venvs.

1. **Read `pyproject.toml`** to identify the project's environment manager and available tasks/environments (look for `[tool.pixi]`, `[tool.hatch]`, `[tool.nox]`, etc.).
2. **For pixi-based projects**:
   - Run `pixi install` (or `pixi install -e <env>` for a specific environment) to ensure the environment is set up.
   - If pixi fails, report it to the user and ask how to proceed — do NOT fall back to manual venvs or pip.
   - Use `pixi run <task>` or `pixi run -e <env> python <script>` to execute code.
   - Look for defined tasks in `pyproject.toml` under `[tool.pixi]`.
3. **Verify the environment works** by running a quick smoke test: `pixi run python -c "import <package_name>"`.
4. If the environment is already set up (`.pixi/envs/` exists with content), skip installation.

**Never:**
- Run `pip install` directly on the system Python
- Create manual virtualenvs as a workaround
- Use `python` or `python3` directly without going through the project's env manager

---

## Phase 1: Full context gathering

Gather **all** available context about this issue. Use parallel agents where possible.

### 1a — Issue itself
- Fetch the full issue body, all comments, labels, assignees, and milestone via `gh issue view <number> --json title,body,comments,labels,assignees,milestone --comments`.
- Note any linked PRs mentioned in the body or comments.

### 1b — Related PRs and discussions
- Search for PRs that reference this issue: `gh pr list --search "<number>" --state all --limit 20`.
- For each related PR, fetch its description, review comments, and status.
- Search for other issues that mention this one: `gh search issues "<number> repo:<owner/repo>"`.

### 1c — Check if already fixed on main
- Look for commits on main that reference this issue: `git log --all --oneline --grep="<number>"`.
- If there are matching commits, read them and assess whether they already address the issue.
- If the issue appears fully fixed on main, **report this finding and stop** — do not proceed to later phases. Ask the user whether they want to close the issue or if there's a remaining aspect to address.

### 1d — Check for related/duplicate issues
- From the search results above, identify any issues that describe the same or overlapping symptoms.
- Summarize any related issues and note whether fixing this one might also resolve them (or conflict with them).

**Deliverable:** Print a structured summary:
- **Issue**: title, core problem, reporter expectations
- **Already fixed?**: yes/no with evidence
- **Related issues**: list with brief notes
- **Related PRs**: list with status and relevance
- **Key constraints**: any API contracts, backward compat concerns, or edge cases mentioned in discussion

---

## Phase 2: Reproduce the issue

Write a **minimal, self-contained test** that demonstrates the bug described in the issue.

### Before writing any reproduction code:

1. **Read existing test files** for the affected module to understand:
   - Import patterns (e.g., accessor registration imports)
   - Fixture usage and test data setup
   - How matplotlib/plotting is configured in tests (e.g., `matplotlib.use("Agg")`, backend setup)
   - Any conftest.py helpers or base classes used
2. **Read the relevant source code** to understand:
   - What imports are needed
   - The exact API signatures involved
   - Any prerequisites for the code path to work

### Writing the reproduction script:

- Place the reproduction script in a temporary file (e.g., `/tmp/repro_issue_<number>.py`).
- **Mirror the import patterns from existing tests exactly** — don't guess at imports.
- Set `matplotlib.use("Agg")` before any matplotlib/plotting imports if testing plot code.
- The test should **fail on the current main branch** to confirm the bug is real.
- Run it using the repo's official tooling (e.g., `pixi run python /tmp/repro_issue_<number>.py`).
- If the issue describes multiple symptoms, write a separate assertion for each.
- If the issue cannot be reproduced (test passes on main), **stop and report** — ask the user for clarification before proceeding.

**Deliverable:** Print:
- The reproduction test code
- The test output showing the failure
- A one-line summary of the confirmed root cause

---

## Phase 3: Comprehensive plan

Now that you understand the issue and can reproduce it, create a detailed implementation plan.

### The plan must cover:
1. **Root cause analysis**: Exactly which code path is wrong and why.
2. **Proposed fix**: Concrete changes to specific files and functions, with rationale.
3. **Edge cases**: Enumerate all edge cases (empty inputs, boundary conditions, type variations, large data, categorical data, sparse data, etc.) and how each is handled.
4. **Downstream impact**: Will this change affect the public API? Any backward compatibility concerns?
5. **Test strategy**: What tests to add or modify, covering the main fix and all edge cases identified.
6. **Risk assessment**: What could go wrong? Any areas of uncertainty?

### Plan format:
Write the plan to `plans/issue-<number>.md` following this structure:

```markdown
# Issue #<number>: <title>

## Root cause
<explanation>

## Proposed changes
| File | Change | Rationale |
|------|--------|-----------|
| ... | ... | ... |

## Edge cases
- [ ] <case 1>: <how handled>
- [ ] <case 2>: <how handled>
...

## Downstream impact
<assessment>

## Test plan
- [ ] Regression test for reported bug
- [ ] <edge case test 1>
- [ ] <edge case test 2>
...

## Risks
- <risk 1>
- <risk 2>
```

**Deliverable:** Print the plan summary and **ask the user for approval before proceeding**.

**STOP HERE AND WAIT FOR USER APPROVAL. Do NOT proceed to Phase 4 until the user explicitly approves.**

---

## Phase 4: Implement the fix

Once the user approves the plan (they may request modifications — incorporate those first):

### 4a — Create a branch (if not already on one)
- If on `main`, create a new branch: `git checkout -b fix/issue-<number>`.
- If already on a feature branch, use it.

### 4b — Implement changes
- Follow the approved plan step by step.
- Keep changes minimal and localized — do not refactor unrelated code.
- After each file change, run the repo's linter/formatter to keep the code clean.

### 4c — Verify against reproduction test
- Re-run the reproduction test from Phase 2. It must now **pass**.
- If it doesn't pass, debug and fix — do not move on until the reproduction test passes.

### 4d — Run the broader test suite
- Run the repo's test suite using official tooling, excluding visual/plot comparison tests that cannot run locally.
- If any existing tests fail, investigate whether the failure is caused by your changes or is pre-existing.
- Fix any regressions introduced by your changes.

### 4e — Lint and format
- Run the repo's full linting/formatting pipeline using official tasks.
- Fix any issues before proceeding.

**Deliverable:** Print a summary of all changes made (files modified, brief description of each change).

---

## Phase 5: Test coverage

### 5a — Check existing test coverage
- Search the test suite for existing tests that cover the affected code path.
- Determine whether any existing test would have caught this bug (and if so, why it didn't).

### 5b — Add regression test
- If no existing test covers the exact bug scenario, add a proper regression test.
- Place it in the appropriate test file following the repo's test organization patterns.
- The test should:
  - Reference the issue number in a comment (e.g., `# Regression test for #<number>`).
  - Fail on the code before the fix and pass after.
  - Cover the main bug scenario AND the edge cases from the plan.
- If a test already exists that covers the scenario, note this and add only the missing edge case tests.

### 5c — Final verification
- Run the updated test suite (excluding visual tests) one final time using official tooling.
- Ensure all tests pass, including the new ones.
- Run linting/formatting one final time.

**Deliverable:** Print:
- Whether existing tests covered this (and if so, which ones)
- New tests added (file, test name, what they cover)
- Final test suite results

---

## Final summary

After all phases complete, print a structured summary:

```
## Issue #<number> — Resolution Summary

**Problem**: <one-sentence description>
**Root cause**: <one-sentence explanation>
**Fix**: <one-sentence description of the change>
**Files changed**: <list>
**Tests added**: <list with descriptions>
**Branch**: <branch name>
**Status**: Ready for review
```
