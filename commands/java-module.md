# /java-module - Create New Maven Module

Create a new feature module following hexagonal architecture.

## Usage

```
/java-module {module-name}
```

## Steps

1. Create module directory: `modules/{module-name}/`

2. Create `pom.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>pl.krupix.nejbors</groupId>
        <artifactId>nejbors-service</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../../pom.xml</relativePath>
    </parent>

    <artifactId>{module-name}</artifactId>
    <name>{Module Name}</name>

    <dependencies>
        <dependency>
            <groupId>pl.krupix.nejbors</groupId>
            <artifactId>common</artifactId>
        </dependency>
    </dependencies>
</project>
```

3. Create package structure:
```
src/main/java/pl/krupix/nejbors/{modulename}/
├── domain/
│   ├── ports/
│   └── query/
├── adapters/
│   ├── web/
│   └── orm/
└── dto/
```

4. Add module to parent `pom.xml`:
```xml
<module>modules/{module-name}</module>
```

5. Add dependency to `app/pom.xml`:
```xml
<dependency>
    <groupId>pl.krupix.nejbors</groupId>
    <artifactId>{module-name}</artifactId>
</dependency>
```

6. Create placeholder files:
   - `domain/{Feature}Facade.java` (interface)
   - `domain/{Feature}FacadeImpl.java`
   - `domain/query/{Feature}Query.java` (interface)
   - `domain/query/{Feature}QueryImpl.java`
   - `adapters/web/{Feature}Controller.java`

## Verification

```bash
./mvnw -pl modules/{module-name} compile
```
