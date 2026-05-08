# /design Command

**Activate with `/design`**

## Purpose

Creates the complete specification for a new feature using the four-file
structure under `.cursor/specs/{feature-name}/`. Decomposes the work into
mini-specs (one per architectural layer) so each task is implementable
and reviewable in isolation.

## Usage

- `/design feature-name` — Create a new feature spec
- `/design @.cursor/specs/feature-name/00-global-spec.md` — Refine an existing spec
- `/design` — Interactive mode (the architect interviews you)

## Persona

Before proceeding, load and adopt the persona defined in
`.cursor/skills/architect/SKILL.md`. Operate within that persona's
boundaries for the entire execution.

## Behavior

1. **Resolve target**:
   - If a feature name is provided, use it (kebab-case).
   - If a path to an existing spec is provided, refine it.
   - Otherwise, ask the user for the feature name and core intent.

2. **Interview if needed** (max 5 questions, only the essentials):
   - Which module(s) does this affect? Existing or new?
   - Is this a Command (write), Query (read), or both?
   - What's the API contract? (endpoint, method, payload shape)
   - Any non-obvious business rules or edge cases?
   - Async via Messenger or sync?

3. **Inspect the codebase** before designing:
   - Look at similar modules under `src/` for patterns to reuse.
   - Note any conventions or shared components (e.g. `App\Shared\Domain\Bus`).
   - Highlight in the plan if you deviate from existing patterns and why.

4. **Read the rules**:
   - `.cursor/rules/backend-standards.mdc` (binding contract)
   - `.cursor/rules/stack.mdc`
   - `.cursor/rules/backend-standards.mdc`
   - `.cursor/rules/documentation-standards.mdc`

5. **Produce the four-file structure** under
   `.cursor/specs/{feature-name}/`:
   - `00-global-spec.md` — What & Why
   - `00-technical-plan.md` — How
   - `tasks/01-domain.md`, `02-application.md`, `03-infrastructure.md`,
     `04-integration.md` (skip layers that don't apply, split if too large)
   - `status.md` — all tasks initially `pending`

6. **Use the templates** in `.cursor/specs/template/` as the starting
   structure for each file.

7. **Stop**. Output a summary with:
   - The file tree created
   - Total estimated effort (sum of task estimates)
   - Any open questions or assumptions made
   - The exact `/build` invocation to start: `/build @.cursor/specs/{feature-name}/tasks/01-domain.md`

## Output structure

```
.cursor/specs/{feature-name}/
├── 00-global-spec.md
├── 00-technical-plan.md
├── tasks/
│   ├── 01-domain.md
│   ├── 02-application.md
│   ├── 03-infrastructure.md
│   └── 04-integration.md
└── status.md
```

## Decomposition rules

Each `tasks/NN-*.md` file is a single-commit unit:

| File | Layer | Typical content | Tests |
|------|-------|-----------------|-------|
| `01-domain.md` | Domain | Entities, VOs, events, exceptions, repository interfaces | PHPUnit |
| `02-application.md` | Application | Commands, Queries, Handlers, Application Services | PHPUnit |
| `03-infrastructure.md` | Infrastructure | Controllers, Payloads, Schemas, Doctrine repos & mappings | PHPUnit + Behat |
| `04-integration.md` | Integration | Behat .feature files, translations, end-to-end verification | Behat |

If a layer is too big for one task (> 2h estimated), split:
- `02a-commands.md` and `02b-queries.md`
- `03a-controllers.md` and `03b-doctrine.md`

If a layer doesn't apply (e.g. read-only feature with no domain changes),
skip it. The numbering can have gaps.

## Hard rules

- NEVER write production PHP code. Skeletons in the technical plan are OK;
  method bodies are not.
- NEVER create a task estimated at more than 2 hours. Split it.
- NEVER skip the architectural checklist from `backend-standards.mdc`.
- ALWAYS write specs in English.
- ALWAYS list translation keys required (per i18n rules in `stack.mdc`).
- ALWAYS reference the existing modules you inspected for patterns.

## Examples

### New feature
```
User: /design member-export-by-status
AI: [adopts architect persona]
[interviews briefly]
[inspects src/ExportProcess/ and src/Member/ for patterns]
[reads rules]
[creates files]
```

Created:
``````
.cursor/specs/member-export-by-status/
├── 00-global-spec.md
├── 00-technical-plan.md
├── tasks/
│   ├── 01-domain.md       (~1h)
│   ├── 02-application.md  (~1.5h)
│   ├── 03-infrastructure.md (~2h)
│   └── 04-integration.md  (~1h)
└── status.md
``````
Estimated effort: ~5.5h total

Notes:
- Reuses ExportProcessor pattern from src/ExportProcess/
- Adds new MemberStatus VO (currently a string)
- Translations needed for: member.export.started, member.export.no_results


Next: /build @.cursor/specs/member-export-by-status/tasks/01-domain.md
### Refining an existing spec
```
User: /design @.cursor/specs/member-export-by-status/00-global-spec.md
AI: [adopts architect persona]
Reviewing existing spec...
The technical plan was missing async dispatch via Messenger.Updated:
- 00-technical-plan.md: added MemberExportRequestedEvent + async handler
- tasks/02-application.md: added EventHandler step
- status.md: 02-application reset to pendingAffected tasks must be re-implemented if previously done.
```
## Integration with the workflow/design feature-name
↓
.cursor/specs/{feature-name}/ created
↓
/build @.cursor/specs/{feature-name}/tasks/01-domain.md
↓
/review
↓
(repeat for each task)