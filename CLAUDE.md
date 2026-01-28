# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Java Spring Boot template repository using **Hexagonal Architecture** (Ports & Adapters) with **CQRS separation**. The project uses Maven and targets Spring Boot applications with PostgreSQL.

**Base package:** `pl.krupix.nejbors`

## Build & Run Commands

```bash
# Build all (skip tests)
./mvnw package -DskipTests

# Build single module
./mvnw -pl modules/{module} package -DskipTests

# Run locally
./mvnw spring-boot:run -pl app -Dspring-boot.run.profiles=local

# Run all tests
./mvnw test

# Run single module tests
./mvnw -pl modules/{module} test

# Run single test class
./mvnw -pl modules/{module} test -Dtest=ClassNameTest

# Run single test method
./mvnw -pl modules/{module} test -Dtest="ClassNameTest#methodName"

# Run with coverage
./mvnw test jacoco:report
```

## Database

```bash
# Start PostgreSQL (Docker)
docker run --name nejbors-db -p 6632:5432 -e POSTGRES_PASSWORD=poiupoiu -d --shm-size=512m postgres

# Check migration status
./mvnw flyway:info -pl app
```

Flyway migrations location: `modules/users/src/main/resources/db/migration/`
Migration naming: `V{yyyyMMddHHmmss}__{description}.sql`

## Architecture

### Module Structure

```
modules/{feature}/src/main/java/pl/krupix/nejbors/{feature}/
├── domain/
│   ├── {Feature}Facade.java          # Write operations interface (port)
│   ├── {Feature}FacadeImpl.java      # Write operations implementation
│   ├── ports/
│   │   └── {Feature}Repository.java  # Data access port interface
│   └── query/
│       ├── {Feature}Query.java       # Read operations interface (port)
│       └── {Feature}QueryImpl.java   # Read operations implementation
├── adapters/
│   ├── web/
│   │   └── {Feature}Controller.java  # REST adapter
│   └── orm/
│       ├── {Feature}Entity.java      # JPA entity
│       ├── {Feature}RepositoryImpl.java  # Port implementation
│       ├── Jpa{Feature}Repository.java   # Spring Data interface
│       └── {Feature}Mapper.java      # MapStruct mapper
├── dto/
│   ├── {Feature}DTO.java
│   ├── Create{Feature}DTO.java
│   └── Update{Feature}DTO.java
└── exception/
    └── {Feature}NotFoundException.java
```

### CQRS Separation

- **Facade** = Write operations (create, update, delete) with `@Transactional`
- **Query** = Read operations (find, get, list) with `@Transactional(readOnly = true)`
- Never mix reads and writes in the same class

### Dependency Direction

```
Controller → Facade/Query (interfaces) → Repository (port) ← RepositoryImpl (adapter)
```

### UUID vs ID

- **External (REST API)**: Always use UUID
- **Internal (database)**: Use Long id
- Never expose database IDs in REST responses

### Cross-Module Communication

Use Spring `ApplicationEvents` for module-to-module communication. Never inject facades from other modules directly.

## Coding Conventions

### Dependency Injection

Constructor injection only via `@RequiredArgsConstructor`. All dependencies must be `final` fields:

```java
@Service
@RequiredArgsConstructor
public class FeatureFacadeImpl implements FeatureFacade {
    private final FeatureRepository featureRepository;
    private final ApplicationEventPublisher eventPublisher;
}
```

### Lombok Usage

- `@RequiredArgsConstructor` for dependency injection
- `@Slf4j` for logging
- `@Builder` for DTOs with many fields
- `@Data` for DTOs

### Mapping

Always use MapStruct for entity-DTO conversion. Never write manual mapping code:

```java
@Mapper(componentModel = "spring")
public interface FeatureMapper {
    FeatureDTO toDto(FeatureEntity entity);
    FeatureEntity toEntity(FeatureDTO dto);
}
```

### Method Size

Max 20 lines per method. Extract complex logic into private methods.

## Testing

### Test Structure

```
src/test/java/pl/krupix/nejbors/{feature}/
├── domain/
│   ├── {Feature}FacadeTest.java
│   └── {Feature}QueryTest.java
└── adapters/web/
    └── {Feature}ControllerTest.java
```

### Test Naming

- Class: `{ClassUnderTest}Test`
- Method: `should{ExpectedBehavior}When{Condition}`

### Test Pattern

```java
@ExtendWith(MockitoExtension.class)
class FeatureFacadeTest {
    @Mock
    private FeatureRepository repository;

    @InjectMocks
    private FeatureFacadeImpl facade;

    @Test
    void shouldCreateFeatureWhenValidInput() {
        // given
        // when
        // then
    }
}
```

### Coverage Target

Minimum 80% line coverage for Facade, Query, and Controller implementations.

## Security

- Get current user via `@AuthenticationPrincipal CustomUserDetails`
- Always verify resource ownership before operations
- Use Bean Validation (`@Valid`, `@NotNull`, `@Size`) at controller level
- Use parameterized queries with `@Param` for native queries

## Banned Patterns

- Field injection (`@Autowired` on fields)
- Exposing entities outside adapters/orm
- Business logic in controllers
- Circular module dependencies
- Manual entity-DTO mapping
- Direct facade injection between modules
