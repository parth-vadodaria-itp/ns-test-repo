---
name: backend-dotnet-aspnet
description: >
  Generates a complete .NET ASP.NET Core Web API. DEFAULTS ONLY — resolved_config takes precedence.
metadata:
  stack: dotnet-aspnet
  version: "2.0"
---

# Backend Skill — .NET / ASP.NET Core

## Section 1 — Metadata & Trigger
**Trigger:** `resolved_config.technology.language` contains ".NET" or "C#" or "csharp" OR `detected_stack = "dotnet-aspnet"`.
**Role:** Defaults only — resolved_config overrides every field in Section 2.

## Section 2 — Defaults (null resolved_config field → use these)

| Field | Default |
|---|---|
| `technology.framework` | ASP.NET Core Web API |
| `technology.version` | .NET 8, C# 12 |
| `technology.orm` | null (in-memory ConcurrentDictionary) |
| `technology.package_manager` | dotnet/NuGet |
| `architecture.pattern` | Layered (Controllers/Services/Models) |
| `architecture.custom_structure` | `[]` → ProjectName/{Controllers,Services,Models}/ |
| `database.primary` | null (in-memory — no DB wiring) |
| `database.driver` | null |
| `api.auth_mechanism` | null (no auth) |
| `docker.port` | 8080 — add `server.port=8080` or `ASPNETCORE_URLS=http://+:8080` |

## Section 3 — Override Protocol

**Database override:**
- `PostgreSQL` → add NuGet: `Npgsql.EntityFrameworkCore.PostgreSQL`; add `AddDbContext<AppDbContext>` in Program.cs; generate `AppDbContext.cs`; connection string: `"Host=localhost;Database=<db>;Username=postgres"`; switch from in-memory to Entity Framework Repository pattern
- `SQL Server` → add `Microsoft.EntityFrameworkCore.SqlServer`; connection string: `"Server=localhost;Database=<db>;Trusted_Connection=True"`
- `SQLite` → add `Microsoft.EntityFrameworkCore.Sqlite`; connection string: `"Data Source=app.db"`
- `In-memory (default)` → keep `ConcurrentDictionary<int, Entity>` + `Interlocked.Increment` — no EF Core

**Auth override:**
- `JWT` → add `Microsoft.AspNetCore.Authentication.JwtBearer`; generate `AuthController.cs`, `JwtSettings.cs`, `TokenService.cs`; configure `AddAuthentication().AddJwtBearer()` in Program.cs; add `[Authorize]` on protected endpoints
- `OAuth2` → add `Microsoft.AspNetCore.Authentication.OAuth`

**ORM override:**
- `EFCore` or `EntityFramework` → add `Microsoft.EntityFrameworkCore`; generate `AppDbContext.cs`; use `DbSet<Entity>` pattern
- `Dapper` → add `Dapper`; use raw SQL via `connection.QueryAsync<Entity>`

**Architecture override:**
- `resolved_config.architecture.custom_structure[]` NOT empty → replace default project layout; adjust namespaces
- `Clean Architecture` → Domain/, Application/, Infrastructure/, API/ projects
- `CQRS` → add `MediatR>=12.0.0`; generate Commands/ and Queries/ folders

**Extra libraries:**
- Merge ALL `resolved_config.technology.extra_libraries[]` as `<PackageReference>` entries in `.csproj`

**Port override:**
- `resolved_config.docker.port` NOT null → set `ASPNETCORE_URLS=http://+:<port>` in Program.cs or appsettings.json; update appsettings.json `"Urls"` key

## Section 4 — Compliance Overlays (additive)

- **HIPAA** → add `AuditLoggingMiddleware.cs`; log all data access without PHI fields; `app.UseMiddleware<AuditLoggingMiddleware>()` before controllers
- **GDPR** → add response header `X-Data-Classification: personal`; no PII in error responses
- **SOC2** → `[Authorize]` on all write endpoints; structured audit log via `ILogger<T>` in services

## Section 5 — Hard Rules (non-overridable)

- NEVER use EF Core, DbContext, or database connections unless DB override explicitly applies
- NEVER include `app.UseHttpsRedirection()` — container runs HTTP only; causes redirect loops
- NEVER use `var` for public API return types — be explicit
- NEVER use TODO, TBD, FIXME, `...`, or empty method bodies `{}`
- NEVER use `[Autowired]` — .NET uses constructor injection
- ALWAYS `ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true` in Docker Alpine images
- ALWAYS `ENV ASPNETCORE_URLS=http://+:<port>` to avoid localhost-only binding
- All namespaces must match folder paths exactly
- ALWAYS derive DLL name from `.csproj` filename — NEVER hardcode "ProjectName"
- Generate as many entity files as the story requires — no cap

---

## Philosophy
Minimum files for the app to run. Zero placeholders. Zero stubs. No empty method bodies.

## Stack (defaults — overridable per Section 2)
.NET 8, ASP.NET Core Web API, C# 12, in-memory collections (no DB by default)

## Default project structure
```
ProjectName/
├── ProjectName.csproj
├── Program.cs
├── appsettings.json
├── Controllers/EntityController.cs
├── Services/IEntityService.cs + EntityService.cs
└── Models/Entity.cs + EntityDto.cs
```
> Override when `resolved_config.architecture.custom_structure[]` is NOT empty.

## Reasoning steps
1. Apply all Section 3 overrides from resolved_config.
2. Extract entities from acceptance_criteria.
3. Map each entity → HTTP methods, URL paths, request/response shapes, status codes.
4. Plan data structures (in-memory or EF Core per override).
5. Generate controller + service interface + service + model + DTO per entity.
6. Self-check: every AC has an endpoint; every endpoint calls a service method.

## File content rules

### ProjectName.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```
Add `<PackageReference>` entries from `extra_libraries[]` and DB/auth overrides.

### Program.cs — `builder.Services.AddControllers()`; NO `UseHttpsRedirection()`; `app.MapControllers()`; register services as `AddSingleton<IEntityService, EntityService>()`.
### Controllers — `[Route("api/v1/[controller]")]`, `[ApiController]`, `ActionResult<T>`, constructor inject service.
### Services — interface + implementation; `ConcurrentDictionary<int, Entity>` + `Interlocked.Increment`; throw `KeyNotFoundException` on not-found.
### Models — plain C# classes, `required` keyword for mandatory props; separate request DTOs from domain models.

## Output format
```json
{
  "status": "success",
  "data": {
    "story_id": "<from story_discovery>",
    "detected_stack": "dotnet-aspnet",
    "resolved_config": {},
    "extracted_requirements": {},
    "generated_files": [
      { "path": "ProjectName/ProjectName.csproj", "content": "..." },
      { "path": "ProjectName/Program.cs", "content": "..." },
      { "path": "ProjectName/appsettings.json", "content": "..." },
      { "path": "ProjectName/Controllers/EntityController.cs", "content": "..." },
      { "path": "ProjectName/Services/IEntityService.cs", "content": "..." },
      { "path": "ProjectName/Services/EntityService.cs", "content": "..." },
      { "path": "ProjectName/Models/Entity.cs", "content": "..." },
      { "path": "ProjectName/Models/EntityDto.cs", "content": "..." }
    ],
    "files_generated": 8,
    "language": "csharp"
  }
}
```