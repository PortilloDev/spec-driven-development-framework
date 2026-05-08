# /refactor Command

**Activate with `/refactor`**

## Purpose

Improves code quality through formatting, static analysis, and architectural refactoring. Ensures code follows PSR-12 standards and hexagonal architecture principles.

## Usage

- `/refactor` - Full refactoring (format + analyze + review)
- `/refactor --fix` - Only code formatting (php-cs-fixer)
- `/refactor --analyze` - Only static analysis (phpstan)
- `/refactor @src/Member/` - Refactor specific module
- `/refactor @src/Member/Application/Command/` - Refactor specific directory

## Commands Reference

| Usage | Commands Executed |
|-------|-------------------|
| `/refactor` | `composer phpcs` + `composer phpstan` + architecture review |
| `/refactor --fix` | `composer phpcs` |
| `/refactor --analyze` | `composer phpstan` |
| `/refactor @path` | Scoped refactoring to path |

## Behavior

When activated with `/refactor`, the AI should:

1. **Code Formatting** (php-cs-fixer):
   ```bash
   composer phpcs
   ```
   - Apply PSR-12 standards
   - Fix spacing and indentation
   - Organize imports
   - Apply consistent syntax

2. **Static Analysis** (phpstan):
   ```bash
   composer phpstan
   ```
   - Detect type errors
   - Find unused code
   - Identify potential bugs
   - Check return types

3. **Architecture Review**:
   - Verify hexagonal architecture compliance
   - Check CQRS pattern usage
   - Identify SOLID violations
   - Suggest improvements

4. **Linter Verification**:
   - Use ReadLints tool
   - Fix any remaining issues

## Quality Checklist

### PSR-12 Standards
- [ ] `declare(strict_types=1);` present
- [ ] Proper namespace declaration
- [ ] One class per file
- [ ] Correct brace placement
- [ ] Consistent spacing
- [ ] No trailing whitespace

### Hexagonal Architecture
- [ ] Domain has no infrastructure dependencies
- [ ] Repository interfaces in Domain
- [ ] Implementations in Infrastructure
- [ ] Controllers only create Commands/Queries
- [ ] No business logic in Controllers

### CQRS Compliance
- [ ] Commands are `readonly class`
- [ ] Queries are `readonly class`
- [ ] Handlers implement `HandlerInterface`
- [ ] Handlers use `__invoke()` method
- [ ] Commands return `Uid` or `void`
- [ ] Queries return data (arrays, DTOs, PaginationResults)

### SOLID Principles
- [ ] **S**ingle Responsibility: One reason to change
- [ ] **O**pen/Closed: Open for extension, closed for modification
- [ ] **L**iskov Substitution: Subtypes are substitutable
- [ ] **I**nterface Segregation: Small, specific interfaces
- [ ] **D**ependency Inversion: Depend on abstractions

## Instructions for AI

When user activates `/refactor`:

1. **Determine scope**:
   - Full codebase or specific path
   - Which tools to run

2. **Run formatting**:
   ```bash
   composer phpcs
   ```
   - Analyze output
   - Report changes made

3. **Run static analysis**:
   ```bash
   composer phpstan
   ```
   - Parse errors/warnings
   - Prioritize by severity

4. **Review architecture** (if full refactor):
   - Check layer dependencies
   - Verify naming conventions
   - Identify patterns violations

5. **Report findings**:
   ```
   Refactoring Report:
   
   📝 Formatting:
   - 12 files formatted
   - PSR-12 compliant ✓
   
   🔍 Static Analysis:
   - 3 issues found (2 errors, 1 warning)
   
   🏗️ Architecture:
   - 1 violation: Controller contains business logic
   
   Suggested fixes:
   1. [Error] Missing return type in MemberHandler::__invoke()
   2. [Error] Unused import in CreateMemberController
   3. [Warning] Consider extracting method in BulkRepository
   4. [Architecture] Move validation logic from Controller to Handler
   ```

6. **Offer to fix**:
   - Ask if user wants automatic fixes
   - Apply fixes following architecture rules

## Common Refactoring Patterns

### Extract Method
When a method is too long or has multiple responsibilities:
```php
// Before
public function __invoke(Command $command): Uid
{
    // 50 lines of code doing multiple things
}

// After
public function __invoke(Command $command): Uid
{
    $this->validateBusinessRules($command);
    $entity = $this->createEntity($command);
    $this->persistAndDispatchEvents($entity);
    return $entity->id();
}
```

### Move to Domain Service
When business logic doesn't belong in Handler:
```php
// Before: Logic in Handler
readonly class CreateMemberCommandHandler implements HandlerInterface
{
    public function __invoke(CreateMemberCommand $command): Uid
    {
        // Complex business calculation here
    }
}

// After: Logic in Domain Service
readonly class CreateMemberCommandHandler implements HandlerInterface
{
    public function __construct(
        private MemberDomainService $domainService,
    ) {}
    
    public function __invoke(CreateMemberCommand $command): Uid
    {
        return $this->domainService->createMember($command);
    }
}
```

### Replace Conditional with Polymorphism
When type checking is done with conditionals:
```php
// Before
if ($type === 'email') {
    $this->sendEmail();
} elseif ($type === 'sms') {
    $this->sendSms();
}

// After
interface NotificationSender {
    public function send(): void;
}
$sender->send();
```

## PHPStan Levels

The project uses phpstan with configuration in `phpstan.neon`:

| Level | Description |
|-------|-------------|
| 0 | Basic checks |
| 1 | + possibly undefined variables |
| 2 | + unknown methods |
| 3 | + return types |
| 4 | + dead code |
| 5 | + argument types |
| 6 | + strict comparison |
| 7 | + union types |
| 8 | + strict rules |

## Integration

- `/refactor` can be run independently
- `/build` automatically runs formatting after each step
- Use before committing to ensure quality
- Combine with `/test` to verify changes don't break functionality

## Examples

### Full refactor
```
User: /refactor

AI executes:
- composer phpcs (fixes 8 files)
- composer phpstan (finds 2 issues)
- Architecture review (1 suggestion)

AI reports and offers fixes
```

### Format only
```
User: /refactor --fix

AI executes:
- composer phpcs

AI reports:
✅ 15 files formatted
✅ PSR-12 compliant
```

### Analyze specific module
```
User: /refactor @src/Bulk/

AI analyzes:
- src/Bulk/ directory only
- Focuses on Bulk module patterns

AI reports:
- BulkActionRepository: Consider query optimization
- BulkController: Extract validation to Handler
```

## Error Resolution

When phpstan finds errors:

1. **Missing return type**:
   ```php
   // Before
   public function find($id)
   
   // After
   public function find(Uid $id): ?Member
   ```

2. **Type mismatch**:
   ```php
   // Error: Parameter $id expects Uid, string given
   // Fix: Cast or validate type
   $member = $this->repo->find(Uid::cast($id));
   ```

3. **Unused code**:
   - Remove unused imports
   - Remove dead methods
   - Clean up unreachable code
