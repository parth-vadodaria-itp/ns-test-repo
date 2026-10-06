---
name: review-java-springboot
description: >
  Code review rules for Java 17 + Spring Boot 3.5.x projects. Covers Spring-specific
  anti-patterns, JPA misuse, REST API correctness, security, and dependency hygiene.
  Activate for detected_stack = "java-springboot" or any Java/Maven diff.
metadata:
  stack: java-springboot
  role: reviewer
  version: "1.0"
---

# Review Skill — Java / Spring Boot

## Trigger
`detected_stack = "java-springboot"` OR diff contains `pom.xml` or `.java` files.

## Rule Set

### SECURITY (prefix: java-sec)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-sec-001 | critical | Hardcoded credentials — any string literal matching regex `(password|secret|apikey|token)\s*=\s*"[^"]{6,}"` |
| java-sec-002 | critical | SQL injection — string concatenation in any query: `"SELECT.*\+` or `entityManager.createNativeQuery("SELECT" + ` |
| java-sec-003 | critical | Missing `@PreAuthorize` or `@Secured` on write endpoints (`@PostMapping`, `@PutMapping`, `@DeleteMapping`) when SecurityConfig is present |
| java-sec-004 | major    | `@CrossOrigin(origins = "*")` on production controllers — should be restricted |
| java-sec-005 | major    | Sensitive data (`password`, `ssn`, `dob`) exposed in `toString()` or JSON response |

### CORRECTNESS (prefix: java-cor)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-cor-001 | major    | POST endpoint returning `ResponseEntity.ok()` (HTTP 200) — should be `ResponseEntity.created(uri).build()` (HTTP 201) |
| java-cor-002 | major    | `findById()` result used without `orElseThrow()` or `isPresent()` check — NPE risk |
| java-cor-003 | major    | `@Transactional` missing on service method that performs >1 DB write |
| java-cor-004 | major    | Exception swallowed in empty `catch` block: `catch (Exception e) {}` |
| java-cor-005 | minor    | Constructor injection replaced with `@Autowired` field injection — violates Section 5 hard rule |
| java-cor-006 | minor    | Magic number literals (untyped `int` / `long` constants) — should be named constants |

### DEPENDENCY (prefix: java-dep)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-dep-001 | critical | Spring Boot parent version < 3.5.x in pom.xml — must be `3.5.14` |
| java-dep-002 | major    | `spring-boot-starter-webmvc` used instead of `spring-boot-starter-web` — wrong for Boot 3.x |
| java-dep-003 | major    | Explicit version tag on a Spring Boot BOM-managed dependency — BOM controls versions |
| java-dep-004 | minor    | `spring-boot-starter-data-jpa` added but no DB driver dependency present |

### PERFORMANCE (prefix: java-perf)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-perf-001 | major    | `findAll()` called inside a loop — N+1 query pattern |
| java-perf-002 | minor    | `@OneToMany` without `fetch = FetchType.LAZY` — defaults to EAGER causing over-fetching |
| java-perf-003 | minor    | Large collection returned directly from controller without pagination |

### MAINTAINABILITY (prefix: java-maint)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-maint-001 | major    | Empty method body `{}` or `throw new UnsupportedOperationException()` — stub left in prod |
| java-maint-002 | minor    | Service class > 300 lines — God class smell; consider splitting by entity |
| java-maint-003 | minor    | DTO used as both request and response type — creates tight coupling |
| java-maint-004 | suggestion | Missing `@Slf4j` or logger on service/controller class |

### GLOBAL EXCEPTION HANDLING (prefix: java-err)

| Rule ID     | Severity | Description |
|-------------|----------|-------------|
| java-err-001 | major    | No `@ControllerAdvice` or `@RestControllerAdvice` present in project — unhandled exceptions return 500 |
| java-err-002 | major    | Exception handler returns `HttpStatus.INTERNAL_SERVER_ERROR` for `NoSuchElementException` — should be 404 |

## Positive patterns (do NOT flag as issues)
- `AtomicLong` + `ConcurrentHashMap` for in-memory storage (correct when no DB override)
- Constructor injection with `final` fields
- `ResponseEntity<?>` return type in controllers
- `@Valid` on `@RequestBody` parameters
- `@Builder` Lombok annotation on model classes

## Quick checklist for Spring Boot diffs
1. Does `pom.xml` have `<version>3.5.14</version>` in parent block?
2. Is the correct starter used (`spring-boot-starter-web`, not `webmvc`)?
3. Do all service methods use constructor injection?
4. Do POST endpoints return 201, not 200?
5. Is `GlobalExceptionHandler.java` present with `@RestControllerAdvice`?
6. Are `findById` results always checked before use?