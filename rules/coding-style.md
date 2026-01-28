# Java Coding Style Rules

## Lombok Usage

Always use:
- `@RequiredArgsConstructor` for dependency injection
- `@Slf4j` for logging
- `@Builder` for DTOs with many fields
- `@Data` for DTOs (getters, setters, equals, hashCode, toString)

## Dependency Injection

Constructor injection only via `@RequiredArgsConstructor`:
```java
@Service
@RequiredArgsConstructor
public class FeatureFacadeImpl implements FeatureFacade {
    private final FeatureRepository featureRepository;
    private final ApplicationEventPublisher eventPublisher;
}
```

All dependencies must be `final` fields.

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Interface (port) | `{Feature}Repository` | `PostRepository` |
| Implementation | `{Feature}RepositoryImpl` | `PostRepositoryImpl` |
| Spring Data | `Jpa{Feature}Repository` | `JpaPostRepository` |
| Write facade | `{Feature}Facade` | `PostFacade` |
| Read query | `{Feature}Query` | `PostQuery` |
| Controller | `{Feature}Controller` | `PostController` |
| Entity | `{Feature}Entity` | `PostEntity` |
| DTO | `{Feature}DTO` | `PostDTO` |
| Mapper | `{Feature}Mapper` | `PostMapper` |

## MapStruct Mappers

Always use MapStruct for entity-DTO conversion:
```java
@Mapper(componentModel = "spring")
public interface FeatureMapper {
    FeatureDTO toDto(FeatureEntity entity);
    FeatureEntity toEntity(FeatureDTO dto);
}
```

Never write manual mapping code.

## Transactions

- `@Transactional` on Facade methods (write operations)
- `@Transactional(readOnly = true)` on Query methods (read operations)

## Method Size

- Max 20 lines per method
- Extract complex logic into private methods
- One public method = one responsibility

## Package Visibility

- Facade/Query interfaces: `public`
- Implementations: package-private when possible
- DTOs: `public`
- Entities: package-private (within adapters/orm)
