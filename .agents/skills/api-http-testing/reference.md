# Reference – API testing notes

## Suggested categories

| Category | Examples |
|---|---|
| `Unit` | Handler/validator tests |
| `Integration` | WebApplicationFactory / Testcontainers |
| `Contract` | Public route smoke |

## Minimal integration assert list

For each write endpoint:

1. `201`/`200` + location/body id when creating
2. `400` on missing required fields
3. `404` on unknown id for get/update/delete
4. Auth failure path when security is enabled

For each read endpoint:

1. Empty list vs non-empty list
2. Filtering/pagination bounds (page size max)

## Isolation tips

- Unique prefixes per test run (GUID) for emails/skus
- Transaction rollback or recreated container for heavy suites
- Never point integration tests at shared production databases
