# /java-feature - Add Feature to Module

Add a new feature (entity, CRUD operations) to an existing module.

## Usage

```
/java-feature {module} {FeatureName}
```

## Steps

1. **Create Entity** in `adapters/orm/`:
```java
@Entity
@Table(name = "{feature_name}s")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class {Feature}Entity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private UUID uuid;

    @Column(nullable = false)
    private String name;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist
    void prePersist() {
        uuid = UUID.randomUUID();
        createdAt = LocalDateTime.now();
    }

    @PreUpdate
    void preUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

2. **Create DTOs** in `dto/`:
   - `{Feature}DTO.java`
   - `Create{Feature}DTO.java`
   - `Update{Feature}DTO.java`

3. **Create Mapper** in `adapters/orm/`:
```java
@Mapper(componentModel = "spring")
public interface {Feature}Mapper {
    {Feature}DTO toDto({Feature}Entity entity);
    {Feature}Entity toEntity({Feature}DTO dto);
    List<{Feature}DTO> toDtoList(List<{Feature}Entity> entities);
}
```

4. **Create Repository Port** in `domain/ports/`:
```java
public interface {Feature}Repository {
    Optional<{Feature}DTO> findByUuid(UUID uuid);
    List<{Feature}DTO> findAll(PageRequest pageRequest);
    {Feature}DTO save({Feature}DTO dto);
    void deleteByUuid(UUID uuid);
}
```

5. **Create JPA Repository** in `adapters/orm/`:
```java
public interface Jpa{Feature}Repository extends JpaRepository<{Feature}Entity, Long> {
    Optional<{Feature}Entity> findByUuid(UUID uuid);
    void deleteByUuid(UUID uuid);
}
```

6. **Create Repository Adapter** in `adapters/orm/`:
```java
@Service
@RequiredArgsConstructor
public class {Feature}RepositoryImpl implements {Feature}Repository {
    private final Jpa{Feature}Repository jpaRepository;
    private final {Feature}Mapper mapper;

    @Override
    public Optional<{Feature}DTO> findByUuid(UUID uuid) {
        return jpaRepository.findByUuid(uuid).map(mapper::toDto);
    }

    @Override
    public {Feature}DTO save({Feature}DTO dto) {
        {Feature}Entity entity = mapper.toEntity(dto);
        return mapper.toDto(jpaRepository.save(entity));
    }
}
```

7. **Create Facade** in `domain/`:
```java
public interface {Feature}Facade {
    {Feature}DTO create(Create{Feature}DTO dto, UUID creatorUuid);
    {Feature}DTO update(UUID uuid, Update{Feature}DTO dto);
    void delete(UUID uuid);
}
```

8. **Create Query** in `domain/query/`:
```java
public interface {Feature}Query {
    Optional<{Feature}DTO> findByUuid(UUID uuid);
    List<{Feature}DTO> findAll(PageRequest pageRequest);
}
```

9. **Create Controller** in `adapters/web/`:
```java
@RestController
@RequestMapping("/{feature}s")
@RequiredArgsConstructor
public class {Feature}Controller {
    private final {Feature}Facade facade;
    private final {Feature}Query query;

    @GetMapping("/{uuid}")
    public ResponseEntity<{Feature}DTO> getByUuid(@PathVariable UUID uuid) {
        return query.findByUuid(uuid)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<{Feature}DTO> create(
            @Valid @RequestBody Create{Feature}DTO dto,
            @AuthenticationPrincipal CustomUserDetails user) {
        return ResponseEntity.ok(facade.create(dto, user.getUuid()));
    }
}
```

10. **Create Flyway Migration** in `modules/users/src/main/resources/db/migration/`:
```sql
-- V{yyyyMMddHHmmss}__create_{feature}_table.sql
CREATE TABLE {feature}s (
    id BIGSERIAL PRIMARY KEY,
    uuid UUID NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE INDEX idx_{feature}s_uuid ON {feature}s(uuid);
```

## Verification

```bash
./mvnw -pl modules/{module} compile
./mvnw -pl modules/{module} test
```
