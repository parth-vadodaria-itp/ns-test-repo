---
name: review-dotnet-aspnet
description: >
  Code review rules for .NET 8 + ASP.NET Core Web API projects. Covers C# 12 patterns,
  EF Core correctness, controller/service separation, HTTPS redirect anti-patterns in
  containers, and dependency injection rules.
  Activate for detected_stack = "dotnet-aspnet" or any .cs / .csproj diff.
metadata:
  stack: dotnet-aspnet
  role: reviewer
  version: "1.0"
---

# Review Skill — .NET / ASP.NET Core

## Trigger
`detected_stack = "dotnet-aspnet"` OR diff contains `.csproj`, `.cs`, or `appsettings.json` files.

## Rule Set

### SECURITY (prefix: cs-sec)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| cs-sec-001 | critical | Connection string with password in `appsettings.json` (not `appsettings.Development.json`) |
| cs-sec-002 | critical | `[AllowAnonymous]` placed on a write action when `[Authorize]` was expected |
| cs-sec-003 | major    | JWT secret hardcoded in `appsettings.json` or `JwtSettings.cs` — must be env/secret |
| cs-sec-004 | major    | Sensitive fields (password hash, SSN) returned directly in `ActionResult<EntityDto>` |

### CORRECTNESS (prefix: cs-cor)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| cs-cor-001 | major    | `app.UseHttpsRedirection()` present — causes redirect loops in containers (port 8080); remove it |
| cs-cor-002 | major    | POST action returning `Ok()` (200) instead of `Created()` or `CreatedAtAction()` (201) |
| cs-cor-003 | major    | `var` used for public API return types in controller actions — be explicit |
| cs-cor-004 | major    | `ConcurrentDictionary` key collision: `_id++` not atomic — use `Interlocked.Increment` |
| cs-cor-005 | major    | Missing null check after `_dict.TryGetValue` — `KeyNotFoundException` thrown directly |
| cs-cor-006 | minor    | `[Autowired]` annotation used — not valid C#; use constructor injection |

### DEPENDENCY (prefix: cs-dep)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| cs-dep-001 | critical | `<TargetFramework>` set to `net7.0` or earlier — must be `net8.0` |
| cs-dep-002 | major    | `Npgsql.EntityFrameworkCore.PostgreSQL` added but `AppDbContext` not registered in `Program.cs` |
| cs-dep-003 | minor    | `<PackageReference>` without a version attribute when not using a BOM — may float to breaking version |

### PERFORMANCE (prefix: cs-perf)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| cs-perf-001 | major    | `GetAll()` returning `IEnumerable<T>` without pagination — unbounded response |
| cs-perf-002 | minor    | `async Task` action method that never actually awaits anything — synchronous work masquerading as async |

### MAINTAINABILITY (prefix: cs-maint)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| cs-maint-001 | major    | Service method with empty body `{ }` or single `throw new NotImplementedException()` |
| cs-maint-002 | minor    | Namespace does not match folder path — C# convention violation |
| cs-maint-003 | minor    | DTO class defined in same file as Controller — keep in separate Models/ folder |
| cs-maint-004 | suggestion | `required` keyword missing on mandatory DTO properties — enables compile-time null safety |

## Positive patterns (do NOT flag)
- `Interlocked.Increment(ref _nextId)` for in-memory ID generation
- Constructor injection via `public EntityService(ILogger<EntityService> logger)`
- `ASPNETCORE_URLS=http://+:8080` in Docker — correct for containerised HTTP
- `ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true` in Alpine images

## Quick checklist for .NET diffs
1. Is `<TargetFramework>net8.0</TargetFramework>` in .csproj?
2. Is `app.UseHttpsRedirection()` absent?
3. Do POST actions return 201?
4. Is `Interlocked.Increment` used for in-memory IDs?
5. Is every service registered via constructor injection in `Program.cs`?