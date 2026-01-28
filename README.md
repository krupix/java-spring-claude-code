# Spring Hexagonal Template

<div align="center">

**Production-ready Spring Boot template with Hexagonal Architecture, optimized for Claude Code**

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3-green.svg)](https://spring.io/projects/spring-boot)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Ready-blueviolet.svg)](https://claude.ai/code)

[Quick Start](#quick-start) · [Architecture](#architecture) · [Security Profiles](#security-profiles) · [Commands](#commands)

</div>

---

## Why This Template?

This template is specifically designed for **AI-assisted development with Claude Code**. It provides:

- **Clear Architecture** — Hexagonal (Ports & Adapters) structure that Claude understands and generates correctly
- **CQRS Pattern** — Separate read/write paths for clean, maintainable code
- **Flexible Security** — Choose JWT, OAuth2, Basic, or no auth during project setup
- **Production Ready** — Docker, CI/CD, observability, and best practices included
- **Comprehensive CLAUDE.md** — Instructions that help Claude Code generate consistent, high-quality code

## Quick Start

### Option 1: Shell Script (Recommended)

```bash
git clone https://github.com/your-org/spring-hexagonal-template.git
cd spring-hexagonal-template
./init.sh
```

You'll be prompted for:
- Project name
- Group ID (e.g., `com.mycompany`)
- Package name
- Security profile (`jwt`, `oauth2`, `basic`, `none`)

### Option 2: Maven Archetype

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

### Run Your Project

```bash
# Start database
docker compose up -d postgres

# Run application
./mvnw spring-boot:run -pl app -Dspring-boot.run.profiles=local
```

## Architecture

```
modules/{feature}/
├── domain/                     # Pure business logic (no frameworks)
│   ├── model/                  # Entities, Value Objects, Aggregates
│   ├── port/                   # Repository interfaces
│   └── event/                  # Domain events
│
├── application/                # Use cases & orchestration
│   ├── command/                # Write operations (CQRS)
│   ├── query/                  # Read operations (CQRS)
│   └── dto/                    # Request/Response objects
│
└── infrastructure/             # Framework implementations
    ├── web/                    # REST controllers
    ├── persistence/            # JPA repositories
    └── messaging/              # Event publishers
```

### Key Principles

| Principle | Implementation |
|-----------|----------------|
| **Dependency Inversion** | Domain defines ports, infrastructure implements them |
| **CQRS** | `CommandHandler` for writes, `QueryHandler` for reads |
| **Domain Events** | Cross-module communication via Spring Events |
| **UUID External** | REST APIs use UUID, database uses Long ID internally |

## Security Profiles

Choose your authentication strategy during project generation:

| Profile | Use Case | Features |
|---------|----------|----------|
| `none` | Internal services, development | No authentication |
| `basic` | Admin panels, simple apps | HTTP Basic + sessions |
| `jwt` | APIs, SPAs, mobile apps | Stateless JWT with refresh tokens |
| `oauth2` | Enterprise SSO | Keycloak, Auth0, Okta integration |

Configure in `application.yml`:

```yaml
app:
  security:
    profile: jwt  # or: none, basic, oauth2
    jwt:
      secret: ${JWT_SECRET}
      access-token-expiry: 15m
      refresh-token-expiry: 7d
```

## Commands

### Build & Run

```bash
./mvnw package -DskipTests          # Build all
./mvnw spring-boot:run -pl app      # Run locally
./mvnw test                         # Run all tests
./mvnw test jacoco:report           # Test with coverage
```

### Single Module

```bash
./mvnw -pl modules/{module} test                        # Test module
./mvnw -pl modules/{module} test -Dtest=ClassNameTest   # Single test class
```

### Docker

```bash
docker compose up -d                # Start all services
docker compose logs -f app          # Follow app logs
docker compose down                 # Stop all services
```

## Working with Claude Code

This template includes a comprehensive `CLAUDE.md` file that instructs Claude Code on:

- Project architecture and conventions
- Where to place new code
- Naming patterns to follow
- Security best practices
- Testing requirements

### Example Prompts

```
> Add a new "products" module with CRUD operations

> Create a domain event when an order is placed

> Add pagination to the users list endpoint

> Write tests for the UserCommandHandler
```

Claude Code will follow the established patterns automatically.

## Project Structure

```
.
├── app/                    # Spring Boot application
│   └── src/main/resources/
│       ├── application.yml
│       └── application-local.yml
├── common/                 # Shared components
│   └── src/main/java/
│       ├── domain/         # Base classes (AggregateRoot, ValueObject)
│       ├── application/    # UseCase, PageRequest, PageResponse
│       └── infrastructure/ # Security, Web, Persistence utilities
├── modules/
│   └── users/              # Example feature module
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── db/migration/           # Flyway migrations
├── CLAUDE.md               # Claude Code instructions
└── README.md
```

## Tech Stack

| Category | Technology |
|----------|------------|
| **Framework** | Spring Boot 3.3, Spring Security, Spring Data JPA |
| **Language** | Java 21 |
| **Database** | PostgreSQL, Flyway migrations |
| **Mapping** | MapStruct |
| **Security** | JWT (jjwt), OAuth2 Resource Server |
| **Testing** | JUnit 5, Mockito, AssertJ |
| **Build** | Maven, Docker |
| **CI/CD** | GitHub Actions |

## Documentation

- [Getting Started Guide](docs/getting-started.md)
- [Architecture Decisions](docs/architecture.md)
- [Security Configuration](docs/security-profiles.md)
- [Creating New Modules](docs/module-guide.md)

## Contributing

Contributions are welcome! Please read our contributing guidelines and submit PRs to the `main` branch.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built for developers who ship fast with AI assistance**

[Report Bug](https://github.com/your-org/spring-hexagonal-template/issues) · [Request Feature](https://github.com/your-org/spring-hexagonal-template/issues)

</div>
