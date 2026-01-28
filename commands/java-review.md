# /java-review - Code Review Checklist

Review Java code for architecture compliance and best practices.

## Architecture Checklist

- [ ] **Hexagonal structure**: domain/, adapters/web/, adapters/orm/, dto/
- [ ] **CQRS separation**: Facade for writes, Query for reads
- [ ] **Ports defined**: Interfaces in domain/ports/
- [ ] **No entity leakage**: Entities stay in adapters/orm/
- [ ] **UUID externally**: REST endpoints use UUID, not Long id
- [ ] **Event-driven**: Cross-module uses ApplicationEvents

## Code Quality Checklist

- [ ] **Constructor injection**: `@RequiredArgsConstructor` with `final` fields
- [ ] **No field injection**: No `@Autowired` on fields
- [ ] **Lombok used**: `@Slf4j`, `@Data`, `@Builder` where appropriate
- [ ] **MapStruct mappers**: No manual entity-DTO conversion
- [ ] **Transactions**: `@Transactional` on write operations
- [ ] **Method size**: Max 20 lines per method

## Security Checklist

- [ ] **Auth check**: `@AuthenticationPrincipal` for user context
- [ ] **Ownership verified**: User can only access their resources
- [ ] **Input validated**: `@Valid` on request bodies
- [ ] **No secrets logged**: Passwords, tokens not in logs
- [ ] **Parameterized queries**: No string concatenation in SQL

## Testing Checklist

- [ ] **Test exists**: Unit test for Facade and Query
- [ ] **Given-When-Then**: Clear test structure
- [ ] **Mocks used**: Dependencies mocked with Mockito
- [ ] **Edge cases**: Null, empty, not found scenarios tested
- [ ] **80% coverage**: Critical paths covered

## Naming Checklist

- [ ] `{Feature}Facade` / `{Feature}FacadeImpl`
- [ ] `{Feature}Query` / `{Feature}QueryImpl`
- [ ] `{Feature}Repository` / `{Feature}RepositoryImpl`
- [ ] `Jpa{Feature}Repository`
- [ ] `{Feature}Entity`
- [ ] `{Feature}DTO`, `Create{Feature}DTO`, `Update{Feature}DTO`
- [ ] `{Feature}Mapper`
- [ ] `{Feature}Controller`

## Common Issues

| Issue | Fix |
|-------|-----|
| Controller calls RepositoryImpl | Inject port interface instead |
| Entity returned from controller | Return DTO, use mapper |
| Database ID in response | Use UUID |
| Logic in controller | Move to Facade |
| Direct module dependency | Use ApplicationEvents |
| Field injection | Convert to constructor injection |
