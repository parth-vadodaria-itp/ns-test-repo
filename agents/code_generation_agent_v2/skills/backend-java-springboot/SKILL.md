---
name: backend-java-springboot
description: >
  Generates a complete Java Spring Boot backend. DEFAULTS ONLY — resolved_config takes precedence.
metadata:
  default-backend: "true"
  stack: java-springboot
  version: "3.0"
---

# Backend Skill — Java / Spring Boot

## Section 1 — Metadata & Trigger
**Trigger:** `resolved_config.technology.language` contains "Java" OR `detected_stack = "java-springboot"` OR null (DEFAULT FALLBACK).
**Role:** Defaults only — resolved_config overrides every field in Section 2.

## Section 2 — Defaults (null resolved_config field → use these)

| Field | Default |
|---|---|
| `technology.framework` | Spring Boot 3.5.14 |
| `technology.version` | Java 17 |
| `technology.orm` | null (in-memory ConcurrentHashMap) |
| `technology.package_manager` | Maven |
| `architecture.pattern` | Layered (controller/service/model) |
| `architecture.custom_structure` | `[]` → src/main/java/.../controller, service, model |
| `database.primary` | null (in-memory — no DB wiring) |
| `database.driver` | null |
| `api.auth_mechanism` | null (no auth) |
| `docker.port` | 8080 |

## Section 3 — Override Protocol

**Database override:**
- `PostgreSQL` → add to pom.xml: `spring-boot-starter-data-jpa`, `postgresql` driver; add `application.properties`: `spring.datasource.url=jdbc:postgresql://localhost:5432/<db>`; switch from in-memory to JPA Repository pattern
- `MySQL` → add: `spring-boot-starter-data-jpa`, `mysql-connector-j`; `spring.datasource.url=jdbc:mysql://localhost:3306/<db>`
- `H2` → add: `spring-boot-starter-data-jpa`, `com.h2database:h2`; `spring.datasource.url=jdbc:h2:mem:testdb`

**Auth override:**
- `JWT` → add `io.jsonwebtoken:jjwt-api:0.12.6`, `jjwt-impl`, `jjwt-jackson`; add `spring-boot-starter-security`; generate `JwtUtil.java`, `JwtFilter.java`, `SecurityConfig.java`, `AuthController.java`
- `OAuth2` → add `spring-boot-starter-oauth2-resource-server`

**Architecture override:**
- `resolved_config.architecture.custom_structure[]` NOT empty → use those as package paths; adjust base_package accordingly
- `hexagonal` → domain/, application/, infrastructure/adapters/ packages
- `microservices` → add Spring Cloud dependencies

**ORM override:**
- `JPA` or `Hibernate` explicitly → add `spring-boot-starter-data-jpa`; switch from in-memory to entity + JpaRepository pattern; add `@Entity`, `@Id`, `@GeneratedValue` on model
- `JOOQ` → add `spring-boot-starter-jooq` + code-gen plugin

**Extra libraries:**
- Merge ALL `resolved_config.technology.extra_libraries[]` into pom.xml `<dependencies>`

**Port override:**
- `resolved_config.docker.port` NOT null → add `server.port=<port>` to `application.properties`

**Migration tool override:**
- `Flyway` → add `spring-boot-starter-flyway`; generate `src/main/resources/db/migration/V1__init.sql`
- `Liquibase` → add `spring-boot-starter-liquibase`

## Section 4 — Compliance Overlays (additive)

- **HIPAA** → add audit logging aspect (`@Aspect`) on all write operations; no PHI in log output
- **GDPR** → add `DataClassificationFilter.java` (response header); `@Transactional` on all data writes
- **SOC2** → all endpoints secured; `@PreAuthorize` on write endpoints; audit log entries

## Section 5 — Hard Rules (non-overridable)

- NEVER use Spring Boot version older than 3.5.x in the parent block — use `3.5.14`
- NEVER use `spring-boot-starter-webmvc` — this is Boot 4 only; use `spring-boot-starter-web`
- NEVER generate `JpaRepository` or datasource config unless DB override explicitly applies
- NEVER use `@Autowired` field injection — constructor injection only
- NEVER use TODO, TBD, FIXME, `...`, or empty method bodies
- NEVER add version tags on Spring Boot managed dependencies (BOM manages them)
- All `package` declarations must match file paths
- Generate 5–15 files based on story complexity

---

## Philosophy
Minimum files for the app to actually run. Zero placeholders. Zero stubs.

## Stack (defaults — overridable per Section 2)
Java 17, **Spring Boot 3.5.14** (latest stable 3.5.x), Lombok, Jakarta Validation, Maven
Persistence: in-memory ConcurrentHashMap (no DB by default)

> **Version note:** Always use `3.5.14`. Do NOT use 3.2 or earlier.
> `spring-boot-starter-web` is correct for Boot 3.x (NOT spring-boot-starter-webmvc).

## Default project structure
```
project-name/
├── pom.xml
└── src/main/java/base_package/
    ├── Application.java
    ├── controller/EntityController.java
    ├── service/EntityService.java
    ├── model/Entity.java
    └── exception/GlobalExceptionHandler.java
```
> Override when `resolved_config.architecture.custom_structure[]` is NOT empty.

## Reasoning steps
1. Apply all Section 3 overrides from resolved_config.
2. Extract entities from acceptance_criteria.
3. Map each entity → HTTP methods, URL paths, request/response shapes, status codes.
4. Plan data structures (in-memory Map or JPA if DB override applies).
5. Generate controller + service + model per entity.
6. Self-check: every AC has an endpoint; every endpoint calls a service method.

## File content rules

### pom.xml (exact parent block)
```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.5.14</version>
  <relativePath/>
</parent>
```
Required deps (no version — BOM manages): `spring-boot-starter-web`, `spring-boot-starter-validation`, `org.projectlombok:lombok`.
Java 17 via `<properties><java.version>17</java.version></properties>`.
Merge `extra_libraries[]` as additional `<dependency>` entries.

### GlobalExceptionHandler.java — handle `MethodArgumentNotValidException` (400), `NoSuchElementException` (404), `Exception` (500).
### Controller — `@RestController`, `/api/v1/resource`, `@Valid`, `ResponseEntity<?>`, constructor inject service.
### Service — `@Service`, constructor injection, `ConcurrentHashMap<Long, Entity>` + `AtomicLong`, throw `NoSuchElementException` on not-found.
### Entity — POJO, Lombok `@Data @NoArgsConstructor @AllArgsConstructor @Builder`, no JPA (unless DB override).

## Output format
```json
{
  "status": "success",
  "data": {
    "story_id": "<from story_discovery>",
    "detected_stack": "java-springboot",
    "resolved_config": {},
    "extracted_requirements": {},
    "generated_files": [
      { "path": "pom.xml", "content": "..." },
      { "path": "src/main/java/.../Application.java", "content": "..." },
      { "path": "src/main/java/.../model/Entity.java", "content": "..." },
      { "path": "src/main/java/.../service/EntityService.java", "content": "..." },
      { "path": "src/main/java/.../controller/EntityController.java", "content": "..." },
      { "path": "src/main/java/.../exception/GlobalExceptionHandler.java", "content": "..." }
    ],
    "files_generated": 6,
    "language": "java"
  }
}
```