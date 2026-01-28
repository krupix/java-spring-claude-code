# Java Testing Skill

## TDD Workflow

1. **RED**: Write a failing test first
2. **GREEN**: Write minimal code to pass
3. **REFACTOR**: Clean up without breaking tests

## Test Structure

```
modules/{feature}/src/test/java/pl/krupix/nejbors/{feature}/
├── domain/
│   ├── {Feature}FacadeTest.java      # Unit tests for facade
│   └── {Feature}QueryTest.java       # Unit tests for query
└── adapters/
    └── web/
        └── {Feature}ControllerTest.java  # Controller tests
```

## Unit Test Pattern

```java
@ExtendWith(MockitoExtension.class)
class FeatureFacadeTest {

    @Mock
    private FeatureRepository featureRepository;

    @Mock
    private ApplicationEventPublisher eventPublisher;

    @InjectMocks
    private FeatureFacadeImpl facade;

    @Test
    void shouldCreateFeature() {
        // given
        CreateFeatureDTO dto = new CreateFeatureDTO("name");
        UUID creatorUuid = UUID.randomUUID();
        FeatureDTO expected = FeatureDTO.builder()
            .uuid(UUID.randomUUID())
            .name("name")
            .build();

        when(featureRepository.save(any())).thenReturn(expected);

        // when
        FeatureDTO result = facade.create(dto, creatorUuid);

        // then
        assertThat(result.getName()).isEqualTo("name");
        verify(featureRepository).save(any());
        verify(eventPublisher).publishEvent(any(FeatureCreatedEvent.class));
    }
}
```

## Controller Test Pattern

```java
@ExtendWith(MockitoExtension.class)
class FeatureControllerTest {

    private MockMvc mockMvc;
    private ObjectMapper objectMapper = new ObjectMapper();

    @Mock
    private FeatureFacade facade;

    @Mock
    private FeatureQuery query;

    @InjectMocks
    private FeatureController controller;

    @BeforeEach
    void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(controller).build();
    }

    @Test
    void shouldReturnFeatureByUuid() throws Exception {
        UUID uuid = UUID.randomUUID();
        FeatureDTO dto = FeatureDTO.builder().uuid(uuid).name("test").build();

        when(query.findByUuid(uuid)).thenReturn(Optional.of(dto));

        mockMvc.perform(get("/features/{uuid}", uuid))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("test"));
    }

    @Test
    void shouldReturn404WhenNotFound() throws Exception {
        UUID uuid = UUID.randomUUID();
        when(query.findByUuid(uuid)).thenReturn(Optional.empty());

        mockMvc.perform(get("/features/{uuid}", uuid))
            .andExpect(status().isNotFound());
    }
}
```

## Test Commands

```bash
# Run all tests
./mvnw test

# Run single module tests
./mvnw -pl modules/{feature} test

# Run specific test class
./mvnw -pl modules/{feature} test -Dtest=FeatureFacadeTest

# Run with coverage
./mvnw test jacoco:report
```

## Naming Conventions

- Test class: `{ClassUnderTest}Test`
- Test method: `should{ExpectedBehavior}When{Condition}`

Examples:
- `shouldCreateFeatureWhenValidInput`
- `shouldThrowExceptionWhenUuidNotFound`
- `shouldReturnEmptyListWhenNoResults`

## Assertions

Use AssertJ for fluent assertions:
```java
assertThat(result).isNotNull();
assertThat(result.getName()).isEqualTo("expected");
assertThat(list).hasSize(3).extracting("name").contains("a", "b", "c");
```

## Coverage Target

Minimum 80% line coverage for:
- Facade implementations
- Query implementations
- Controllers
