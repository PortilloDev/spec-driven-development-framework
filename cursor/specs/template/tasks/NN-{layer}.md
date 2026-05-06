# Task NN: {Layer Name}

## Metadata

- **Layer**: Domain | Application | Infrastructure | Integration
- **Estimated effort**: ~{X}h (must be ≤ 2h)
- **Depends on**: {previous task numbers, or "none"}
- **Status**: pending | in progress | ready for review | done | blocked

## Goal

{One paragraph: what this task delivers and why it's a coherent unit.}

## Files to create

- `src/{Module}/{Layer}/.../{ClassName}.php`
- ...

## Files to modify

- `src/{Module}/{Layer}/.../{ExistingClass}.php` — {what changes}

## Implementation checklist

- [ ] {Concrete step 1}
- [ ] {Concrete step 2}
- [ ] {Concrete step 3}
- [ ] All architectural checks from `backend-standards.mdc` pass for this layer

## Tests required

### PHPUnit

- `tests/{Module}/{Layer}/{ClassName}Test.php`:
    - `testHappyPath` — {what it asserts}
    - `testRejectsInvalidInput` — {what it asserts}
    - `testThrowsExpectedException` — {what it asserts}

### Behat (if Infrastructure or Integration)

- `features/admin/{module}/{action}.feature`:
    - Scenario: {happy path description}
    - Scenario: {edge case description}

## Definition of Done

- [ ] All implementation checklist items completed
- [ ] All listed tests added and passing
- [ ] `composer tests` green for this layer
- [ ] `composer behat` green (if applicable)
- [ ] `composer phpcs` clean
- [ ] No deviation from `backend-standards.mdc` checklist
- [ ] `status.md` updated to "ready for review"

## Notes for the developer

{Anything specific the architect wants the developer to know — gotchas,
existing patterns to follow, places where to look for similar code.}