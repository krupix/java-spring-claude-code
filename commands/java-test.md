# /java-test - TDD Workflow

Test-driven development workflow for Java.

## Usage

```
/java-test {module} {ClassName}
```

## TDD Cycle

### 1. RED - Write Failing Test

Create test class in `src/test/java/.../`:
```java
@ExtendWith(MockitoExtension.class)
class {ClassName}Test {

    @Mock
    private DependencyOne depOne;

    @InjectMocks
    private {ClassName} sut;  // system under test

    @Test
    void should{Behavior}When{Condition}() {
        // given
        var input = ...;
        when(depOne.method()).thenReturn(...);

        // when
        var result = sut.methodUnderTest(input);

        // then
        assertThat(result).isEqualTo(expected);
        verify(depOne).method();
    }
}
```

Run and confirm failure:
```bash
./mvnw -pl modules/{module} test -Dtest={ClassName}Test
```

### 2. GREEN - Implement Minimal Code

Write just enough code to pass the test:
```java
@Service
@RequiredArgsConstructor
public class {ClassName} {
    private final DependencyOne depOne;

    public ReturnType methodUnderTest(InputType input) {
        // minimal implementation
        return result;
    }
}
```

Run and confirm pass:
```bash
./mvnw -pl modules/{module} test -Dtest={ClassName}Test
```

### 3. REFACTOR - Clean Up

- Extract methods for clarity
- Remove duplication
- Improve naming
- Keep tests green

```bash
./mvnw -pl modules/{module} test
```

## Test Commands

```bash
# All tests in module
./mvnw -pl modules/{module} test

# Single test class
./mvnw -pl modules/{module} test -Dtest={ClassName}Test

# Single test method
./mvnw -pl modules/{module} test -Dtest="{ClassName}Test#should{Behavior}When{Condition}"

# All tests with coverage
./mvnw test jacoco:report

# Skip tests in build
./mvnw package -DskipTests
```

## Test Patterns

### Given-When-Then
```java
@Test
void shouldReturnUserWhenUuidExists() {
    // given - setup
    UUID uuid = UUID.randomUUID();
    UserDTO expected = UserDTO.builder().uuid(uuid).build();
    when(repository.findByUuid(uuid)).thenReturn(Optional.of(expected));

    // when - action
    Optional<UserDTO> result = query.findByUuid(uuid);

    // then - verification
    assertThat(result).isPresent();
    assertThat(result.get().getUuid()).isEqualTo(uuid);
}
```

### Exception Testing
```java
@Test
void shouldThrowExceptionWhenNotFound() {
    UUID uuid = UUID.randomUUID();
    when(repository.findByUuid(uuid)).thenReturn(Optional.empty());

    assertThatThrownBy(() -> facade.getByUuid(uuid))
        .isInstanceOf(FeatureNotFoundException.class)
        .hasMessageContaining(uuid.toString());
}
```

### Controller Testing
```java
@Test
void shouldReturn201OnCreate() throws Exception {
    CreateDTO dto = new CreateDTO("name");
    FeatureDTO created = FeatureDTO.builder().uuid(UUID.randomUUID()).build();

    when(facade.create(any(), any())).thenReturn(created);

    mockMvc.perform(post("/features")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(dto)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.uuid").exists());
}
```

## Coverage Target

- Facade: 80%+
- Query: 80%+
- Controller: 80%+
- Repository adapters: integration tests
