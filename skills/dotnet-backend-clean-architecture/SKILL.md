---
name: dotnet-backend-clean-architecture
description: Clean Architecture + CQRS (MediatR) for .NET 10 APIs. Vertical-slice Features with Commands/Queries, handlers, validators, repositories, FluentMigrator, Refit. Use when creating or refactoring .NET APIs, endpoints, commands, queries, entities, or when the user mentions Clean Architecture, CQRS, or MediatR.
---

# Backend .NET – Clean Architecture + CQRS

Guide for creating and maintaining .NET 10 backends with Clean Architecture, CQRS (MediatR), and **Features** organization (vertical slice).

## When to use

- Add a new **feature** (new entity + CRUD or operations).
- Add a **Command** or **Query** to an existing feature.
- Create **Controller**, **Handler**, **Validator**, **Repository**, or **UnitOfWork**.
- Configure **HTTP integration** (Refit + Http.Resilience).
- Review or refactor code to follow this skill's pattern.

## When not to use

- Projects that do not use CQRS or MediatR.
- Simple minimal APIs without layered architecture.
- Codebases that use EF Migrations instead of FluentMigrator.
- Projects that already follow a different, established backend pattern.

## Target stack

| Technology | Usage |
|---|---|
| .NET 10 | Runtime and SDK (`net10.0`, SDK 10.0.301+) |
| MediatR 14 | CQRS |
| FluentValidation 12 | Validation in handler |
| EF Core 10 | ORM (InMemory / SQL Server) |
| FluentMigrator 8 | SQL schema (not EF Migrations) |
| Refit 11 + Http.Resilience 10 | HTTP integrations |
| xUnit + NSubstitute + FluentAssertions | Unit tests |

Solution at backend root: `{Solution}.sln` and `.editorconfig` (adjust names to your project).

## Adaptation

- Replace `{Prefix}` / `{Solution}` with the target solution naming.
- If the project uses **EF Migrations** instead of FluentMigrator, skip Migration checklist items and keep EF maps consistent with migrations.
- Keep layer boundaries even when folder or project names differ.
- Example starter that follows this shape: [CleanStack](https://github.com/luismpenholato/clean-stack) — optional, not required.

## Solution structure

```txt
{solution-root}/
├── {Solution}.sln
├── .editorconfig
├── {Prefix}.Interface/          # API (Controllers, Program, Middlewares)
├── {Prefix}.Application/        # Features/ → Commands and Queries
├── {Prefix}.Domain/             # Entities, Interfaces, Services
├── {Prefix}.Infrastructure/     # EF Core, Repositories, UnitOfWork
├── {Prefix}.CrossCutting/       # DTOs, Options, Helpers
├── {Prefix}.CrossCutting.IOC/   # ConfigureBindings* (DI)
├── {Prefix}.Migration/          # FluentMigrator (console)
└── {Prefix}.Tests/              # Features/ mirroring Application
```

Dependency rule: **Interface** → Application, CrossCutting, CrossCutting.IOC. **Application** and **Infrastructure** reference **Domain**. **Application** does not reference Infrastructure.

## Checklist – New feature (e.g., Orders)

**Domain**

- [ ] `Entities/Order.cs` implementing `IEntity<TKey>` (e.g., `long`).
- [ ] `Interfaces/Repositories/IOrderRepository.cs` extending `ICrudRepository<Order, long>`.
- [ ] `Interfaces/UnitOfWork/IOrderUnitOfWork.cs` with repository + `SaveChangesAsync`.

**CrossCutting**

- [ ] `Dto/Orders/OrderDto.cs` (and other API DTOs if needed).

**Infrastructure**

- [ ] `Persistence/Map/OrderEntityConfig.cs` (EF Fluent API).
- [ ] `DbSet<Order>` in `AppDbContext`.
- [ ] `Repositories/OrderRepository.cs` extending `RepositoryBase<Order, long>`.
- [ ] `UnitOfWork/OrderUnitOfWork.cs` implementing `IOrderUnitOfWork`.
- [ ] **CrossCutting.IOC:** register in `ConfigureBindingsRepository` and `ConfigureBindingsUnitOfWork`.

**Application – Commands**

- [ ] Folder `Features/Orders/Commands/CreateOrder/`.
- [ ] `CreateOrderCommand.cs`: `record` with `: IRequest<OrderDto>`.
- [ ] `CreateOrderHandler.cs`: inject `IOrderUnitOfWork` and `IValidator<T>`; `ValidateAndThrowAsync` at the start; return DTO.
- [ ] `CreateOrderValidator.cs`: `AbstractValidator<CreateOrderCommand>` when validation is needed.

**Application – Queries**

- [ ] Folder `Features/Orders/Queries/ListOrders/`.
- [ ] `ListOrdersQuery.cs` + `ListOrdersHandler.cs`.

**Interface**

- [ ] `Controllers/OrdersController.cs`: `[ApiController]`, `IMediator`, HTTP returns (Ok, CreatedAtAction, NotFound, NoContent).

**Tests**

- [ ] In `{Prefix}.Tests/Features/Orders/` mirroring Application:
  - `{Command}HandlerTests.cs` / `{Query}HandlerTests.cs` — NSubstitute; mock `IValidator<T>` in handlers.
  - `{Command}ValidatorTests.cs` — real validator + `Validate()`.
- [ ] `[Trait("Category", "Orders")]`.

**Migration**

- [ ] New FluentMigrator class in `{Prefix}.Migration/Migrations/` (e.g., `Mig_YYYYMMDDHHMMSS_CreateOrders.cs`).
- [ ] Keep aligned with EF `Persistence/Map/`.

## External HTTP integration (Products pattern)

When a feature consumes an external API:

1. **CrossCutting:** integration DTOs (e.g., `ExternalProductDto`), `Options/ExternalProductApiOptions.cs`.
2. **Domain:** `Interfaces/Integration/IExternalProductIntegration.cs` (Refit) and `Interfaces/Services/IExternalProductService.cs`.
3. **Domain/Services:** implementation that translates Refit exceptions (e.g., 404 → null).
4. **CrossCutting.IOC:** `ConfigureBindingsIntegration` — `AddRefitClient<T>()` + `AddStandardResilienceHandler()`.
5. **Application:** query or command using `IExternalProductService`.

Config: `ExternalProductApi:BaseUrl` in appsettings.

## Code patterns

**Command (record)**

```csharp
public sealed record CreateProductCommand(
    string? Name,
    string? Description,
    decimal? Price,
    bool IsActive = true,
    int? ImportFromExternalId = null) : IRequest<ProductDto>;
```

**Handler (explicit validation — no global ValidationBehavior)**

```csharp
public async Task<ProductDto> Handle(CreateProductCommand request, CancellationToken cancellationToken)
{
    await _validator.ValidateAndThrowAsync(request, cancellationToken);
    // logic + repository via UoW
}
```

**Validator registration (IOC)**

```csharp
services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();
```

**Controller**

- One controller per resource; async methods with `CancellationToken`; only `Mediator.Send`.

## Error handling

- FluentValidation → `ValidationException` → global middleware → 400 ProblemDetails.
- Business rules → `InvalidOperationException` → 400.
- Other exceptions → 500 (generic message in production).
- Middleware: `GlobalExceptionMiddleware` in **Interface**.

## Anti-patterns

- Global `ValidationBehavior` instead of explicit `ValidateAndThrowAsync` in handlers.
- Application referencing Infrastructure directly.
- EF Migrations mixed with FluentMigrator in the same project.
- Business logic in controllers instead of MediatR handlers.
- Skipping validator or handler unit tests for new commands.

## Conventions

- Namespaces: `{Prefix}.Application.Features.{Feature}.Commands.{CommandName}` and `...Queries.{QueryName}`.
- One command/query **per folder** in Commands/ or Queries/.
- API DTOs in **CrossCutting.Dto**; entities in Domain.
- New bindings in **CrossCutting.IOC** (`ConfigureBindings*.cs`).
- **Do not use** `Application/Common/Behaviors/ValidationBehavior` in this pattern — validation stays in the handler.
- Formatting: `dotnet format {Solution}.sln` (`.editorconfig`).

## Related

- Method / agent workflow: [Verdicspec](https://github.com/luismpenholato/verdicspec)
- Detailed structure, endpoints, and flows: [reference.md](reference.md)
