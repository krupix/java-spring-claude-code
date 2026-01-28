# Java Security Rules

## Authentication

Get current user via `@AuthenticationPrincipal`:
```java
@GetMapping("/me")
public ResponseEntity<UserDTO> getCurrentUser(
    @AuthenticationPrincipal CustomUserDetails userDetails) {
    return ResponseEntity.ok(userQuery.findByUuid(userDetails.getUuid()));
}
```

Never trust client-provided user IDs for authorization checks.

## Authorization

Always verify resource ownership:
```java
public void deletePost(UUID postUuid, UUID requestingUserUuid) {
    PostDTO post = postRepository.findByUuid(postUuid)
        .orElseThrow(() -> new PostNotFoundException(postUuid));

    if (!post.getAuthorUuid().equals(requestingUserUuid)) {
        throw new AccessDeniedException("Not the author");
    }
    // proceed with deletion
}
```

## Input Validation

- Use Bean Validation (`@Valid`, `@NotNull`, `@Size`, etc.)
- Validate at controller level
- Never trust client input

```java
@PostMapping
public ResponseEntity<FeatureDTO> create(
    @Valid @RequestBody CreateFeatureDTO dto) {
    // dto is already validated
}
```

## SQL Injection Prevention

- Always use parameterized queries
- Spring Data JPA methods are safe by default
- For native queries, use `@Param`:

```java
@Query("SELECT e FROM Entity e WHERE e.name = :name")
List<Entity> findByName(@Param("name") String name);
```

## Sensitive Data

- Never log passwords, tokens, or PII
- Never return passwords in DTOs
- Use `@JsonIgnore` for sensitive fields

## File Uploads

- Validate file types and sizes
- Store files outside web root
- Generate random filenames (UUID)
