# Technical Plan: {Feature Name}

## API Contract

### Endpoint
```
{HTTP_METHOD} /api/v1/{path}
```
### Request

**Headers**:
- `Authorization: Bearer {jwt}`
- `Content-Type: application/json`

**Body** (`{Verb}{Noun}Payload`):
```json
{
  "field": "type"
}
```

**Validation**:
- `field`: required, type, constraints

### Response

**Success** ({status_code}):
```json
{
  "field": "value"
}
```

**Errors**:
- `400` — Validation failed
- `404` — Resource not found
- `409` — Conflict

## CQRS classification

- **Operation type**: Command | Query | Both
- **Sync or async**: sync (default) | async (`AsyncMessageInterface`)
- **Returns**: `Uid` | `void` | DTO | `PaginationResults`

## Domain model changes

### New

- `{Aggregate}` — {responsibility}
- `{ValueObject}` — {responsibility}
- `{DomainEvent}` — emitted when {trigger}
- `{Exception}` — thrown when {trigger}

### Modified

- `{ExistingClass}` — {what changes and why}

## Module structure

``` 
src/{Module}/
├── Domain/
│   ├── Contracts/
│   │   └── {Entity}RepositoryInterface.php   (new | modified)
│   ├── Model/
│   │   └── {Aggregate}.php                   (new | modified)
│   ├── Event/
│   │   └── {Event}.php                       (new)
│   └── Exception/
│       └── {Exception}.php                   (new)
├── Application/
│   ├── Command/{Action}/
│   │   ├── {Action}Command.php               (new)
│   │   └── {Action}CommandHandler.php        (new)
│   └── Query/{Name}/
│       ├── {Name}Query.php                   (new)
│       └── {Name}QueryHandler.php            (new)
└── Infrastructure/
├── Doctrine/
│   ├── Mapping/{Entity}.orm.xml          (new | modified)
│   └── Persistence/Doctrine{Entity}Repository.php  (new | modified)
└── Symfony/
├── Controller/{Action}{Entity}Controller.php   (new)
└── Http/
├── Request/{Action}{Entity}Payload.php     (new)
└── Response/{Entity}Schema.php             (new | modified)
```
## Doctrine mapping changes

- File: `src/{Module}/Infrastructure/Doctrine/Mapping/{Entity}.orm.xml`
- New fields: {list}
- New relations: {list}
- Migration: run `bin/console doctrine:migrations:diff` after task 03

## Messenger routing

- `{CommandClass}` — sync (default) | async (implements `AsyncMessageInterface`)
- Middleware applied: CacheMiddleware | LockMiddleware | none

## Dependencies

- **Modules consumed**: {e.g. App\Shared, App\ExportProcess}
- **Patterns reused**: {reference to existing module that's being mirrored}
- **Deviations from existing patterns**: {none | explanation if any}

## Testing strategy

| Layer | Test type | Location |
|-------|-----------|----------|
| Domain | PHPUnit | `tests/{Module}/Domain/` |
| Application | PHPUnit | `tests/{Module}/Application/` |
| Infrastructure | PHPUnit + Behat | `tests/{Module}/Infrastructure/` + `features/admin/{module}/` |

## Out of scope (deferred to other features)

- {Things the architect noticed but explicitly excluded from this feature}