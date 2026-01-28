# Hexagonal Architecture Skill

## When to Use

When creating new features, modules, or refactoring existing code to follow hexagonal architecture patterns.

## Module Creation Checklist

1. [ ] Create Maven module with proper parent reference
2. [ ] Create package structure: `domain/`, `adapters/web/`, `adapters/orm/`, `dto/`
3. [ ] Define port interfaces in `domain/ports/`
4. [ ] Create Facade (write) and Query (read) interfaces
5. [ ] Implement adapters for each port
6. [ ] Create MapStruct mappers
7. [ ] Add module to parent POM

## Package Template

```
modules/{feature}/
├── pom.xml
└── src/
    ├── main/java/pl/krupix/nejbors/{feature}/
    │   ├── domain/
    │   │   ├── {Feature}Facade.java
    │   │   ├── {Feature}FacadeImpl.java
    │   │   ├── ports/
    │   │   │   └── {Feature}Repository.java
    │   │   └── query/
    │   │       ├── {Feature}Query.java
    │   │       └── {Feature}QueryImpl.java
    │   ├── adapters/
    │   │   ├── web/
    │   │   │   └── {Feature}Controller.java
    │   │   └── orm/
    │   │       ├── {Feature}Entity.java
    │   │       ├── {Feature}RepositoryImpl.java
    │   │       ├── Jpa{Feature}Repository.java
    │   │       └── {Feature}Mapper.java
    │   └── dto/
    │       ├── {Feature}DTO.java
    │       └── Create{Feature}DTO.java
    └── test/java/pl/krupix/nejbors/{feature}/
        └── domain/
            └── {Feature}FacadeTest.java
```

## Port Interface Pattern

```java
// Read port
public interface FeatureQuery {
    Optional<FeatureDTO> findByUuid(UUID uuid);
    List<FeatureDTO> findAll(PageRequest pageRequest);
}

// Write port
public interface FeatureFacade {
    FeatureDTO create(CreateFeatureDTO dto, UUID creatorUuid);
    FeatureDTO update(UUID uuid, UpdateFeatureDTO dto);
    void delete(UUID uuid);
}

// Data access port
public interface FeatureRepository {
    Optional<FeatureDTO> findByUuid(UUID uuid);
    FeatureDTO save(FeatureDTO dto);
    void deleteByUuid(UUID uuid);
}
```

## Adapter Implementation Pattern

```java
@Service
@RequiredArgsConstructor
public class FeatureRepositoryImpl implements FeatureRepository {
    private final JpaFeatureRepository jpaRepository;
    private final FeatureMapper mapper;

    @Override
    public Optional<FeatureDTO> findByUuid(UUID uuid) {
        return jpaRepository.findByUuid(uuid)
            .map(mapper::toDto);
    }

    @Override
    public FeatureDTO save(FeatureDTO dto) {
        FeatureEntity entity = mapper.toEntity(dto);
        return mapper.toDto(jpaRepository.save(entity));
    }
}
```

## Dependency Rules

```
✓ Controller → Facade (interface)
✓ Controller → Query (interface)
✓ FacadeImpl → Repository (interface)
✓ RepositoryImpl → JpaRepository
✓ RepositoryImpl → Mapper

✗ Controller → RepositoryImpl
✗ Domain → Entity
✗ Any layer → Concrete implementations
```
