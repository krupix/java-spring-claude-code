# /java-migration - Create Flyway Migration

Create a new database migration.

## Usage

```
/java-migration {description}
```

## Location

All migrations go to: `modules/users/src/main/resources/db/migration/`

## Naming Convention

```
V{yyyyMMddHHmmss}__{description}.sql
```

Example: `V20250128143000__create_bookings_table.sql`

## Generate Filename

```bash
echo "V$(date +%Y%m%d%H%M%S)__description.sql"
```

## Common Migration Templates

### Create Table
```sql
CREATE TABLE {table_name} (
    id BIGSERIAL PRIMARY KEY,
    uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP,
    deleted_at TIMESTAMP  -- soft delete
);

CREATE INDEX idx_{table_name}_uuid ON {table_name}(uuid);
CREATE INDEX idx_{table_name}_created_at ON {table_name}(created_at);
```

### Add Column
```sql
ALTER TABLE {table_name}
ADD COLUMN {column_name} VARCHAR(255);
```

### Add Column with Default
```sql
ALTER TABLE {table_name}
ADD COLUMN {column_name} BOOLEAN DEFAULT false NOT NULL;
```

### Add Foreign Key
```sql
ALTER TABLE {table_name}
ADD COLUMN {foreign}_id BIGINT REFERENCES {foreign_table}(id);

CREATE INDEX idx_{table_name}_{foreign}_id ON {table_name}({foreign}_id);
```

### Add UUID Foreign Key
```sql
ALTER TABLE {table_name}
ADD COLUMN {foreign}_uuid UUID;

CREATE INDEX idx_{table_name}_{foreign}_uuid ON {table_name}({foreign}_uuid);
```

### Create Join Table
```sql
CREATE TABLE {table_a}_{table_b} (
    {table_a}_id BIGINT NOT NULL REFERENCES {table_a}(id) ON DELETE CASCADE,
    {table_b}_id BIGINT NOT NULL REFERENCES {table_b}(id) ON DELETE CASCADE,
    PRIMARY KEY ({table_a}_id, {table_b}_id)
);
```

### Add Enum Type
```sql
CREATE TYPE {type_name} AS ENUM ('VALUE_A', 'VALUE_B', 'VALUE_C');

ALTER TABLE {table_name}
ADD COLUMN status {type_name} DEFAULT 'VALUE_A';
```

## Verification

```bash
# Check migration status
./mvnw flyway:info -pl app

# Run migrations
./mvnw spring-boot:run -pl app -Dspring-boot.run.profiles=local
```

## Rules

- Never modify existing migrations
- Always add new migrations for changes
- Test locally before committing
- Use descriptive names
