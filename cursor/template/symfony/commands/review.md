# /review Command

**Activate with `/review`**

## Purpose

Audits the most recently implemented mini-spec. Runs all quality gates,
verifies architectural compliance, and either approves with a commit
message suggestion or returns a precise list of changes required.

## Usage

- `/review` — Reviews the last mini-spec marked "ready for review" in status.md
- `/review @.cursor/specs/{feature}/tasks/NN-{layer}.md` — Review a specific task
- `/review --commit` — On approval, also create the conventional commit

## Persona

Before proceeding, load and adopt the persona defined in
`.cursor/skills/senior-reviewer/SKILL.md`. Operate within that persona's
boundaries for the entire execution.

## Behavior

When activated:

1. **Resolve target task**:
    - If a path is provided, use it.
    - Otherwise, read `.cursor/specs/{feature}/status.md` and pick the most
      recent task marked `ready for review`.
    - If none, ask the user which task to review.

2. **Load context**:
    - The mini-spec file
    - The global spec and technical plan in the same `{feature}/` folder
    - `.cursor/rules/backend-standards.mdc`, `stack.mdc`, `backend-standards.mdc`
    - The git diff vs HEAD~1 for the modified files

3. **Run quality gates** (in order):
```bash
   composer tests
   composer behat
   composer phpcs
   composer phpstan
```
Capture all output.

4. **Walk the verification matrix** defined in the reviewer skill.

5. **Produce the review report** following the skill's template.

6. **Apply the decision**:
    - APPROVED:
        - Update `status.md`: this task → `- [x] NN-{layer} — done`
        - If `--commit` was passed: create the conventional commit
        - Suggest the next task in the feature folder
    - CHANGES REQUESTED:
        - Update `status.md`: this task → `- [~] NN-{layer} — changes requested`
        - Output the actionable list. Do NOT auto-fix — that's the developer's job
          on the next `/build` cycle.

## Iteration model

The expected loop is:

/build → /review → (changes? /build again, fixing only what review flagged) → /review → APPROVED → next task


The reviewer persona MUST NOT make code changes. It can only flag and suggest.
This boundary is what makes the iteration meaningful — single-agent self-fixing
loses the value of having a separate review pass.

## Hard rules

- NEVER skip running the test suites. The decision is data-driven.
- NEVER approve a task if the architectural checklist has any failing item.
- NEVER touch source code files. Only `status.md` and (optionally) git.
- NEVER advance to the next mini-spec automatically — the developer command
  picks tasks, not the reviewer.

## Examples

### Standard review

User: /review
AI: [adopts senior reviewer persona]
Reviewing: .cursor/specs/bulk-conflict-detection/tasks/02-application.md

Running gates...
✓ PHPUnit: 12/12 pass
✓ Behat: skipped (Application layer)
✓ phpcs: clean
✗ phpstan: 1 issue

Architecture checklist: 4/5 pass
✗ Handler uses concrete repository instead of interface

Decision: CHANGES REQUESTED

Issues (blocker):
1. src/Bulk/Application/Command/.../Handler.php:23
   Inject BulkRepositoryInterface, not DoctrineBulkRepository.
   The handler must depend on the abstraction.
2. src/Bulk/Application/Command/.../Handler.php:41
   phpstan: $member may be null. Add a guard or use ->orFail().

status.md updated: 02-application — changes requested

### Approved review with commit

User: /review --commit
AI: [reviews, all green]
Decision: APPROVED

Commit created:
feat(bulk): add conflict detection command handler

Implements .cursor/specs/bulk-conflict-detection/tasks/02-application.md

status.md updated: 02-application — done
Next pending: 03-infrastructure