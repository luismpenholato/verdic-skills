---
name: api-http-testing
description: Design and implement HTTP API tests for .NET (xUnit) and Node/TypeScript APIs — unit handlers, integration with Testcontainers, contract checks. Use when adding endpoints, fixing regressions, or when the user asks for API tests, integration tests, or coverage of Commands/Queries.
---

# API HTTP testing

Guide for testing REST APIs with a clear pyramid: fast unit tests for handlers/validators, integration tests for real HTTP + persistence, and light contract checks.

## When to use

- New or changed Commands, Queries, controllers/endpoints.
- Regression bugs on API behavior (status codes, validation, auth failures).
- User asks for unit, integration, or API contract tests.

## When not to use

- Pure UI component tests (use the frontend skills).
- Load/performance testing as the primary goal.
- Testing third-party SaaS APIs you do not own (mock at the boundary instead).

## Test pyramid

| Layer | Scope | Speed | Tools (examples) |
|---|---|---|---|
| Unit | Handler + validator logic | Fast | xUnit, NSubstitute, FluentAssertions / Vitest |
| Integration | HTTP pipeline + DB | Medium | WebApplicationFactory, Testcontainers |
| Contract / smoke | Status + shape of public routes | Medium | HttpClient, Playwright API, Newman |

Prefer many unit tests, fewer integration tests, minimal flaky E2E.

## Checklist – New endpoint / feature

- [ ] Happy path returns expected status and body shape.
- [ ] Validation failures → `400` (ProblemDetails or project equivalent).
- [ ] Not found → `404`; conflict/business rule → documented status.
- [ ] Unauthorized/forbidden covered when auth exists (`401`/`403`).
- [ ] Handler unit tests with mocked UnitOfWork/repositories.
- [ ] Validator tests with real validator + `Validate()` / `safeParse`.
- [ ] At least one integration test hitting the real route when persistence matters.

## .NET patterns

**Handler unit test**

- Mock `IValidator<T>` and UnitOfWork/repositories with NSubstitute.
- Assert `ValidateAndThrowAsync` is invoked when that is the project rule.
- Use `[Trait("Category", "{Feature}")]` for filtering.

**Integration**

- `WebApplicationFactory<Program>` + Testcontainers for SQL when InMemory is insufficient.
- Prefer authenticating test clients the same way production will (header/cookie).
- Reset or isolate DB state between tests.

## TypeScript / Node patterns

- Integration tests against a disposable Postgres (Testcontainers) when the API owns schema.
- Share Zod/OpenAPI contracts between client and tests when available.
- Avoid testing framework internals; assert HTTP status + JSON fields that matter.

## Anti-patterns

- Only testing controllers by newing them without the HTTP pipeline when middleware matters.
- Huge E2E suites for every field validation (put that in unit/validator tests).
- Sharing mutable static DB state across tests without cleanup.
- Asserting full JSON snapshots that break on every DTO rename.

## Related

- Backend structure: `dotnet-backend-clean-architecture`
- Security of test data/secrets: `application-security-review`
- Examples and tables: [reference.md](reference.md)
