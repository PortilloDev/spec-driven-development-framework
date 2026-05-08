# /build Command

**Activate with `/build`**

## Purpose

Implements one mini-spec from `.cursor/specs/{feature}/tasks/NN-*.md`
following strict TDD. Stops at task boundary for review. Does NOT
auto-advance to the next task.

## Usage

- `/build @.cursor/specs/{feature}/tasks/NN-{layer}.md` — Implement a specific task
- `/build` — Pick the first `pending` task from `status.md` (asks for feature if ambiguous)
- `/build --continue` — Resume the most recently `in-progress` task

## Persona

Before proceeding, load and adopt the persona defined in
`.cursor/skills/developer/SKILL.md`. Operate within that persona's
boundaries for the entire execution.

## Behavior

1. **Resolve target task**:
   - If a path is provided, use it.
   - Otherwise, read `status.md` from the active feature folder and pick
     the first `pending` task in numerical order.
   - If multiple features have pending tasks, ask the user.

2. **Update status to in-progress**:
   In `.cursor/specs/{feature}/status.md`, mark the chosen task as
   `- [~] NN-{layer} — in progress`.

3. **Load context** (in order):
   - The mini-spec file
   - `00-global-spec.md` and `00-technical-plan.md` from the same folder
   - `.cursor/rules/backend-standards.mdc`
   - `.cursor/rules/stack.mdc` and `backend-standards.mdc`
   - The current state of `src/` for any files mentioned in the task

4. **Apply TDD cycle** for every public class or method in the task:
   - **Red**: Write the test first. Run it. It must fail.
   - **Green**: Write minimum code to pass. Run the test. It must pass.
   - **Refactor**: Clean up. Run `composer phpcs`.
   - **Re-run all relevant tests**: must still pass.

5. **Test execution by layer touched** (from `stack.mdc`):
   | Layer | Tests to run |
   |-------|--------------|
   | Domain | `composer tests` |
   | Application | `composer tests` |
   | Infrastructure | `composer tests` + `composer behat` |
   | Controller / API | always `composer behat` |

   Tests run inside the Docker container.

6. **Walk the task checklist**: every item must be done or explicitly
   marked as deferred (with reason).

7. **Final quality pass**:
   - `composer phpcs` — must be clean
   - `composer phpstan` — report any new issues but don't block (the
     reviewer will gate on this)

8. **Update status**:
   In `status.md`, change the task line to:
   `- [x] NN-{layer} — ready for review`

9. **Stop and report**. DO NOT pick the next task. DO NOT commit.

## Report format

After completing a task, output:

```
Task: {feature}/{NN}-{layer}.md — ready for review
Files created:

{full paths}

Files modified:

{full paths}

Tests added:

tests/{...}/SomeTest.php → testCreatesMemberSuccessfully
tests/{...}/SomeTest.php → testThrowsWhenOrganizationNotFound
features/admin/member/create.feature → 3 scenarios

Test results:

composer tests: {pass/total}
composer behat: {pass/total} (or "skipped, layer is Application")
composer phpcs: clean
composer phpstan: {clean | N issues — see notes}

Notes:

{anything that surprised you, deviated, or needs the reviewer's attention}

Next step: /review
```

## Hard rules

- NEVER write production code without a failing test first.
- NEVER mark a task as ready for review if any test is red.
- NEVER auto-advance to the next task after completion.
- NEVER make a commit. The reviewer (with `--commit`) handles that.
- NEVER deviate from `backend-standards.mdc`. If the task forces a deviation,
  STOP, mark the task as `blocked`, and ask the architect to amend.
- NEVER touch files outside the scope of the current task.

## What to do when stuck

If during implementation you hit one of these, STOP and report:

| Situation | Action |
|-----------|--------|
| Spec is unclear or contradictory | Mark task `blocked: needs architect`, output the contradiction |
| Implementation needs files outside the task scope | Mark task `blocked: scope conflict`, list the extra files |
| A test reveals the design is wrong | Mark task `blocked: design issue`, describe the problem |
| External dependency missing | Mark task `blocked: dependency`, name the dependency |

In all cases, update `status.md` to `- [!] NN-{layer} — blocked: {reason}`
and stop. Do NOT improvise.

## Examples

### Standard build

```
User: /build @.cursor/specs/bulk-conflict-detection/tasks/01-domain.md
AI: [adopts developer persona]
[reads spec, plan, rules]
[marks task as in-progress]
```
Implementing 01-domain.md (Domain layer):

Step 1: BulkConflict value object
- Wrote test: tests/Bulk/Domain/Model/BulkConflictTest.php → testRejectsEmptyConflictList
- Test failed (expected). Wrote minimum code. Test passes.
- phpcs clean.

Step 2: BulkConflictDetectionService
- Wrote test: tests/Bulk/Domain/Service/BulkConflictDetectionServiceTest.php → 4 tests
- All red, then all green. phpcs clean.

Step 3: BulkConflictRepositoryInterface in Contracts/
- No test (interface). Documented in PHPDoc.

Final pass:
- composer tests: 87/87 pass
- composer phpcs: clean
- composer phpstan: clean

status.md updated: 01-domain — ready for review.

Next step: /review

### Blocked task

```
User: /build @.cursor/specs/member-export/tasks/02-application.md
AI: [adopts developer persona]
[reads spec, plan]
```

BLOCKED: scope conflict.

Task 02-application.md requires modifying
src/ExportProcess/Domain/ExportProcessor.php to accept a new strategy.
That file is in another module and was not listed as a dependency in
the technical plan.

status.md updated: 02-application — blocked: scope conflict.

Recommend: /design @.cursor/specs/member-export/00-technical-plan.md
to amend the dependency list, then split into:
- 02a-export-strategy-extension.md (touches ExportProcess module)
- 02b-member-export-handler.md (Member module only)

## Integration with the workflow
```
/build @.cursor/specs/{feature}/tasks/NN-{layer}.md
↓
[TDD: red → green → refactor]
↓
status.md updated: ready for review
↓
/review
↓
APPROVED → next /build NN+1
CHANGES → /build same task again, fixing only what was flagged
```