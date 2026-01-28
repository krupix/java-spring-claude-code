# /java-build - Build Commands

Quick reference for Maven build commands.

## Build Commands

```bash
# Build all (skip tests - fastest)
./mvnw package -DskipTests

# Build all with tests
./mvnw package

# Clean build
./mvnw clean package -DskipTests

# Build single module
./mvnw -pl modules/{module} package -DskipTests

# Build module and its dependencies
./mvnw -pl modules/{module} -am package -DskipTests

# Install to local repo
./mvnw install -DskipTests
```

## Run Commands

```bash
# Run locally
./mvnw spring-boot:run -pl app -Dspring-boot.run.profiles=local

# Run with specific profile
./mvnw spring-boot:run -pl app -Dspring-boot.run.profiles=preprod

# Run built JAR
java -jar app/target/app-1.0.0-SNAPSHOT.jar --spring.profiles.active=local
```

## Test Commands

```bash
# All tests
./mvnw test

# Single module
./mvnw -pl modules/{module} test

# Single class
./mvnw -pl modules/{module} test -Dtest=ClassNameTest

# Single method
./mvnw -pl modules/{module} test -Dtest="ClassNameTest#methodName"

# With coverage
./mvnw test jacoco:report
```

## Database

```bash
# Start PostgreSQL (Docker)
docker run --name nejbors-db -p 6632:5432 \
  -e POSTGRES_PASSWORD=poiupoiu -d \
  --shm-size=512m postgres

# Flyway migrate (runs automatically on app start)
./mvnw flyway:migrate -pl app

# Flyway info
./mvnw flyway:info -pl app
```

## Dependency Management

```bash
# Check for updates
./mvnw versions:display-dependency-updates

# Dependency tree
./mvnw dependency:tree

# Analyze unused dependencies
./mvnw dependency:analyze
```

## Common Issues

| Issue | Solution |
|-------|----------|
| Module not found | Add to parent pom.xml `<modules>` |
| Class not found | Add dependency to module pom.xml |
| Tests fail | Run with `-X` for debug output |
| Port in use | Check for running processes on 8080 |
