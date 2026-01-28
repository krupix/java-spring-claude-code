# Spring Hexagonal Template - Redesign Plan

## Overview

Transform the current Java Spring repository from a personal template with hardcoded values into a professional, public open-source template following hexagonal architecture best practices.

## Goals

1. Remove all private/personal information (krupix, nejbors)
2. Add placeholder tokens for project generation
3. Fix architectural issues (DTOs in wrong layer, missing domain model, etc.)
4. Add Maven archetype + shell script for project generation
5. Add multiple security profiles (none, basic, jwt, oauth2)
6. Add production-ready infrastructure (Docker, CI/CD, observability)
7. Create comprehensive common module with base patterns

## Architecture Changes

### Current (Problematic)

```
modules/{feature}/
├── domain/
│   ├── {Feature}Facade.java          # Confusing name
│   ├── {Feature}FacadeImpl.java
│   ├── ports/
│   │   └── {Feature}Repository.java  # Returns DTOs (wrong!)
│   └── query/
├── adapters/
│   ├── web/
│   └── orm/
├── dto/                               # Wrong location
└── exception/
```

### Proposed (Correct Hexagonal)

```
modules/{feature}/
├── domain/                            # Pure business logic
│   ├── model/
│   │   ├── {Feature}.java             # Rich domain entity
│   │   └── {Feature}Id.java           # Value object
│   ├── event/
│   │   └── {Feature}CreatedEvent.java
│   ├── exception/
│   │   └── {Feature}NotFoundException.java
│   └── port/
│       ├── {Feature}Repository.java   # Returns domain model
│       └── {Feature}EventPublisher.java
│
├── application/                       # Use cases
│   ├── command/
│   │   ├── Create{Feature}Command.java
│   │   └── {Feature}CommandHandler.java
│   ├── query/
│   │   ├── {Feature}QueryHandler.java
│   │   └── {Feature}View.java
│   └── dto/
│       ├── {Feature}Response.java
│       └── Create{Feature}Request.java
│
└── infrastructure/                    # Framework adapters
    ├── web/
    │   └── {Feature}Controller.java
    ├── persistence/
    │   ├── {Feature}Entity.java
    │   ├── {Feature}JpaRepository.java
    │   ├── {Feature}RepositoryAdapter.java
    │   └── {Feature}Mapper.java
    └── messaging/
        └── {Feature}EventPublisherAdapter.java
```

## Placeholder System

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `${groupId}` | Maven group ID | `com.example` |
| `${artifactId}` | Maven artifact ID | `my-service` |
| `${package}` | Java base package | `com.example.myservice` |
| `${packagePath}` | Directory path | `com/example/myservice` |
| `${securityProfile}` | Security type | `jwt` |

## Security Profiles

### Profile: none
- No authentication
- Use case: internal services, development

### Profile: basic
- HTTP Basic + sessions
- Use case: simple apps, admin panels

### Profile: jwt
- Stateless JWT tokens
- Includes: login, refresh, token validation
- Use case: APIs, SPAs, mobile apps

### Profile: oauth2
- OAuth2 Resource Server
- Integrates with: Keycloak, Auth0, Okta
- Use case: enterprise SSO

## Common Module Components

### Domain Layer
- `AggregateRoot<ID>` - Base for aggregates with domain events
- `DomainEvent` / `BaseDomainEvent` - Event interfaces
- `ValueObject` - Base for value objects
- `NotFoundException` - Base exception

### Application Layer
- `UseCase<I, O>` - Generic use case interface
- `CommandHandler<C, R>` - Write operation handler
- `QueryHandler<Q, R>` - Read operation handler
- `PageRequest` / `PageResponse` - Pagination

### Infrastructure Layer
- `GlobalExceptionHandler` - Unified error handling
- `ApiError` - Standard error response
- `TracingFilter` - Request tracing (X-Trace-Id)
- `BaseEntity` - JPA base with UUID + audit fields
- `DomainEventPublisher` - Transactional event dispatch

## Project Generation

### Maven Archetype
```bash
mvn archetype:generate \
  -DarchetypeGroupId=io.github.spring-hexagonal \
  -DarchetypeArtifactId=spring-hexagonal-archetype \
  -DarchetypeVersion=1.0.0 \
  -DgroupId=com.mycompany \
  -DartifactId=my-service \
  -Dpackage=com.mycompany.myservice \
  -DsecurityProfile=jwt
```

### Shell Script
```bash
./init.sh
# Interactive prompts for all values
```

## Infrastructure

### Docker
- Multi-stage Dockerfile (JDK 21)
- docker-compose.yml with PostgreSQL
- Health checks configured

### CI/CD (GitHub Actions)
- Build and test on PR
- Checkstyle linting
- OWASP dependency check
- JaCoCo coverage to Codecov

### Observability
- Spring Actuator endpoints
- Prometheus metrics
- JSON structured logging (production)
- Request tracing via MDC

## Files to Delete

- `rules/architecture.md`
- `rules/coding-style.md`
- `rules/security.md`
- `commands/java-build.md`
- `commands/java-test.md`
- `commands/java-feature.md`
- `commands/java-module.md`
- `commands/java-migration.md`
- `commands/java-review.md`
- `skills/hexagonal-architecture/`
- `skills/java-testing/`
- `rules/` directory
- `commands/` directory
- `skills/` directory

## Files to Create

### Root
- `README.md`
- `LICENSE` (MIT)
- `.gitignore`
- `.editorconfig`
- `init.sh`

### Documentation
- `docs/getting-started.md`
- `docs/architecture.md`
- `docs/security-profiles.md`
- `docs/module-guide.md`

### Archetype
- `archetype/pom.xml`
- `archetype/src/main/resources/META-INF/maven/archetype-metadata.xml`
- `archetype/src/main/resources/archetype-resources/...`

### Template
- `template/pom.xml`
- `template/app/...`
- `template/common/...`
- `template/modules/users/...` (example)
- `template/docker/...`
- `template/db/migration/...`

### CI/CD
- `.github/workflows/ci.yml`
- `.github/workflows/release.yml`
- `.github/ISSUE_TEMPLATE/...`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/dependabot.yml`

## Implementation Order

1. Delete old files (rules/, commands/, skills/)
2. Create directory structure
3. Create common module base classes
4. Create example users module
5. Create app module with security profiles
6. Create Docker and CI/CD files
7. Create archetype configuration
8. Create init.sh script
9. Update CLAUDE.md
10. Create README.md and documentation
11. Test archetype generation

## Success Criteria

- [ ] No personal information in codebase
- [ ] `./init.sh` generates working project
- [ ] `mvn archetype:generate` generates working project
- [ ] All security profiles work correctly
- [ ] Docker compose starts successfully
- [ ] CI pipeline passes
- [ ] Generated project compiles and tests pass
