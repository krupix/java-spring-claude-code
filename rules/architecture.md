# Java Hexagonal Architecture Rules

## Module Structure

Every feature module follows this structure:
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
│   └── {Feature}DTO.java
└── exception/
    └── {Feature}NotFoundException.java
```

## CQRS Separation

- **Facade** = Write operations (create, update, delete)
- **Query** = Read operations (find, get, list)
- Never mix reads and writes in the same class

## Dependency Direction

```
Controller → Facade/Query → Repository (Port) ← RepositoryImpl (Adapter)
```

- Domain depends on nothing external
- Adapters implement domain ports
- Controllers only call Facade or Query interfaces

## UUID vs ID

- **External (REST API)**: Always use UUID
- **Internal (database)**: Use Long id
- Never expose database IDs in REST responses

## Database Migrations

All Flyway migrations go to: `modules/users/src/main/resources/db/migration/`

Naming: `V{yyyyMMddHHmmss}__{description}.sql`

## Cross-Module Communication

Use Spring ApplicationEvents for module-to-module communication:
```java
applicationEventPublisher.publishEvent(new FeatureCreatedEvent(uuid));
```

Never inject facades from other modules directly.

## Banned Patterns

- No field injection (`@Autowired` on fields)
- No exposing entities outside adapters/orm
- No business logic in controllers
- No circular module dependencies
