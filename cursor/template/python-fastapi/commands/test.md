# /test Command

**Activate with `/test`**

## Purpose

Executes tests independently from the build process. Useful for verifying code quality, running specific test suites, or debugging test failures.

## Usage

- `/test` - Run all tests (PHPUnit + Behat)
- `/test unit` - Run only PHPUnit tests
- `/test feature` - Run only Behat tests
- `/test @features/admin/bulk/` - Run specific Behat feature directory
- `/test @features/admin/bulk/list.feature` - Run specific feature file
- `/test @src/Member/` - Run PHPUnit tests for specific module

## Test Commands Reference

| Usage | Command Executed |
|-------|------------------|
| `/test` | `composer tests && composer behat` |
| `/test unit` | `composer tests` |
| `/test feature` | `composer behat` |
| `/test @features/X.feature` | `composer behat -- features/X.feature` |
| `/test --stop` | `composer behat --stop-on-failure` |

## Behavior

When activated with `/test`, the AI should:

1. **Identify test scope** from arguments
2. **Execute appropriate test command(s)**
3. **Analyze results**:
   - If pass: Report success
   - If fail: Analyze errors and suggest fixes
4. **Provide detailed report**

## Test Suites

### PHPUnit (Unit Tests)
```bash
# Run all unit tests
composer tests

# Or directly
bin/phpunit
```

**Location**: `tests/`
**Purpose**: Test Domain entities, Value Objects, Handlers in isolation

### Behat (Functional Tests)
```bash
# Run all Behat tests
composer behat

# Run with stop on failure
vendor/bin/behat --stop-on-failure

# Run specific feature file
vendor/bin/behat features/admin/member/create.feature

# Run specific scenario by line
vendor/bin/behat features/admin/member/create.feature:15
```

**Location**: `features/`
**Purpose**: Test API endpoints, full request/response cycles

## Directory Structure

```
tests/
├── Unit/
│   └── {Module}/
│       ├── Application/
│       │   └── Command/
│       │       └── {Handler}Test.php
│       └── Domain/
│           └── Model/
│               └── {Entity}Test.php
└── Behat/
    └── Context/
        └── {Context}Context.php

features/
├── admin/
│   ├── member/
│   │   ├── create.feature
│   │   └── list.feature
│   └── bulk/
│       └── actions.feature
├── member/
│   └── profile.feature
└── anonymous/
    └── login.feature
```

## Instructions for AI

When user activates `/test`:

1. **Parse arguments** to determine scope:
   - No args: Run all tests
   - `unit`: PHPUnit only
   - `feature`: Behat only
   - Path: Specific test file/directory

2. **Execute tests**:
   ```bash
   # All tests
   composer tests
   composer behat
   
   # Unit only
   composer tests
   
   # Feature only
   composer behat
   
   # Specific
   vendor/bin/behat features/admin/member/create.feature
   ```

3. **Analyze output**:
   - Parse test results
   - Identify failures
   - Extract error messages

4. **Report results**:
   ```
   Test Results:
   ✅ PHPUnit: 45 tests, 120 assertions, 0 failures
   ✅ Behat: 23 scenarios, 89 steps, 0 failures
   
   All tests passing!
   ```

5. **If failures exist**:
   ```
   Test Results:
   ❌ PHPUnit: 45 tests, 118 assertions, 2 failures
   
   Failures:
   1. MemberTest::testCreateWithInvalidEmail
      - Expected: InvalidEmailException
      - Got: No exception thrown
      - File: tests/Unit/Member/Domain/Model/MemberTest.php:45
   
   Suggested fix:
   - Add email validation in Member::create() method
   ```

## Test Types Reference

### Unit Tests (PHPUnit)
Test in isolation:
- Domain entities and behavior
- Value Objects validation
- Command/Query Handlers (with mocked repositories)
- Domain Services

### Functional Tests (Behat)
Test full flow:
- API endpoints
- Request validation
- Response format
- Authentication/Authorization
- Database state changes

## Common Behat Steps

```gherkin
# Authentication
Given I am authenticated as "admin@example.com"
Given I am authenticated as "member@example.com"

# HTTP Requests
When I add "Content-Type" header equal to "application/json"
When I send a "POST" request to "/api/v1/members" with body:
"""
{
  "email": "test@example.com"
}
"""

# Response Assertions
Then the response status code should be 201
And the JSON should be valid according to this schema:
"""
{
  "id": "string"
}
"""
And the JSON node "data.email" should be equal to "test@example.com"
```

## Error Analysis

When tests fail, the AI should:

1. **Read the full error output**
2. **Identify the type of failure**:
   - Assertion failure: Expected vs Actual mismatch
   - Exception: Unexpected error thrown
   - Database: Constraint violation, missing data
   - HTTP: Wrong status code, invalid response
3. **Locate the source**:
   - Test file and line number
   - Source file being tested
4. **Suggest specific fix**:
   - Code change needed
   - Missing implementation
   - Test assertion correction

## Integration with Workflow

- `/test` can be run independently at any time
- `/build` automatically runs `/test` after each step
- Use `/test` to verify before committing
- Use `/test @specific/path` to debug specific issues

## Examples

### Run all tests
```
User: /test

AI executes:
- composer tests
- composer behat

AI reports:
✅ All 68 tests passing
✅ All 45 scenarios passing
```

### Run specific feature
```
User: /test @features/admin/bulk/list.feature

AI executes:
- vendor/bin/behat features/admin/bulk/list.feature

AI reports:
✅ 3 scenarios passing
✅ 15 steps executed
```

### Debug failing test
```
User: /test unit

AI executes:
- composer tests

AI reports:
❌ 1 failure in MemberHandlerTest

Analysis:
- Test expects MemberCreatedException but handler returns Uid
- The handler should throw exception when member already exists

Suggested fix:
Add duplicate check in CreateMemberCommandHandler before save
```
