# Database Schema Declaration

Cycle/Database provides a powerful declarative approach to define and manage database table structures, foreign keys,
and indexes. The schema declaration system compares your desired schema with the current database state and
automatically generates the necessary SQL operations.

## Principle of Work

The schema declaration system operates through a comparison-based approach:

1. **Load Current State**: DBAL reads the existing table structure from the database
2. **Normalize Format**: The structure is normalized into an internal representation
3. **Compare Schemas**: Your declared schema is compared with the existing one
4. **Generate Operations**: SQL commands are generated based on the differences
5. **Execute Changes**: The operations are executed when `save()` is called

This declarative approach means you define *what* you want the schema to look like, not *how* to change it. DBAL handles
the transformation automatically.

> **Note**  
> The schema system can work alongside external migration systems. See the [Migrations](/docs/en/database/migrations.md)
> documentation for integration details.

## Getting Started

To work with table schemas, obtain an `AbstractTable` instance from your database:

```php
use Cycle\Database\Database;

$database = new Database(/* ... */);

// Get schema for new or existing table
$schema = $database->table('users')->getSchema();

// Check if table exists in database
if ($schema->exists()) {
    echo "Table exists";
} else {
    echo "Table will be created";
}
```

> **Note**  
> You don't need to check for table existence before working with the schema. The system handles both creation and
> modification transparently.

## Column Declaration

### Basic Column Definition

Add columns to your schema using fluent method calls:

```php
$schema = $database->table('users')->getSchema();

// Method 1: Explicit column() call
$schema->column('id')->primary();
$schema->column('email')->string(255);
$schema->column('balance')->decimal(10, 2);

// Method 2: Direct shortcut (recommended)
$schema->primary('id');
$schema->string('email', 255);
$schema->decimal('balance', 10, 2);

// Save changes to database
$schema->save();
```

Both approaches are equivalent, but shortcuts are more concise for simple declarations.

### Abstract Types

Cycle DBAL uses abstract types that map to appropriate database-specific column types. This ensures your schema works
across different database systems.

**Type Mapping Example:**

```php
// This declaration:
$schema->string('username', 64);

// Generates in MySQL:
// `username` VARCHAR(64) NULL

// Generates in PostgreSQL:
// "username" CHARACTER VARYING(64) NULL

// Generates in SQLite:
// "username" TEXT NULL
```

### Column Type Reference

#### Primary Keys

| Type             | Method                  | Description                                          | Example                       |
|------------------|-------------------------|------------------------------------------------------|-------------------------------|
| **primary**      | `primary($column)`      | Auto-incrementing integer primary key (32-bit)       | `$schema->primary('id')`      |
| **bigPrimary**   | `bigPrimary($column)`   | Auto-incrementing big integer primary key (64-bit)   | `$schema->bigPrimary('id')`   |
| **smallPrimary** | `smallPrimary($column)` | Auto-incrementing small integer primary key (16-bit) | `$schema->smallPrimary('id')` |

**Usage Notes:**

- Only one auto-increment column per table
- Automatically added to table's primary key index
- Use `bigPrimary` for tables expecting billions of records

#### Integer Types

| Type             | Method                  | Description               | Typical Range                                           | Example                               |
|------------------|-------------------------|---------------------------|---------------------------------------------------------|---------------------------------------|
| **integer**      | `integer($column)`      | Standard integer (32-bit) | -2,147,483,648 to 2,147,483,647                         | `$schema->integer('age')`             |
| **tinyInteger**  | `tinyInteger($column)`  | Small integer (8-bit)     | -128 to 127 (or 0 to 255 unsigned)                      | `$schema->tinyInteger('status_code')` |
| **smallInteger** | `smallInteger($column)` | Small integer (16-bit)    | -32,768 to 32,767                                       | `$schema->smallInteger('port')`       |
| **bigInteger**   | `bigInteger($column)`   | Large integer (64-bit)    | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | `$schema->bigInteger('file_size')`    |

**Practical Examples:**

```php
// User age (0-150)
$schema->tinyInteger('age')->nullable(false);

// Order quantity (thousands)
$schema->integer('quantity')->defaultValue(1);

// File size in bytes (gigabytes)
$schema->bigInteger('file_size_bytes')->nullable(false);
```

#### String Types

| Type           | Method                     | Parameters              | Max Size            | Description            | Example                          |
|----------------|----------------------------|-------------------------|---------------------|------------------------|----------------------------------|
| **string**     | `string($column, $length)` | `length` (default: 255) | 255-65,535*         | Variable-length string | `$schema->string('email', 255)`  |
| **text**       | `text($column)`            | None                    | 65,535 bytes        | Standard text field    | `$schema->text('description')`   |
| **tinyText**   | `tinyText($column)`        | None                    | 255 bytes           | Very short text        | `$schema->tinyText('code')`      |
| **mediumText** | `mediumText($column)`      | None                    | 16,777,215 bytes    | Medium text field      | `$schema->mediumText('article')` |
| **longText**   | `longText($column)`        | None                    | 4,294,967,295 bytes | Very large text        | `$schema->longText('content')`   |

*Maximum string length varies by database system.

**Best Practices:**

```php
// Use string for indexable fields (emails, usernames)
$schema->string('email', 255)->unique();
$schema->string('username', 64)->nullable(false);

// Use text types for non-indexable content
$schema->text('bio');
$schema->longText('blog_post_content');

// Short codes or identifiers
$schema->string('country_code', 2); // 'US', 'UK', etc.
$schema->string('currency', 3);      // 'USD', 'EUR', etc.
```

> **Note**  
> You cannot add indexes to `text`, `mediumText`, or `longText` columns in most databases. Use `string` for fields that
> need indexing.

#### Numeric Types

| Type        | Method                                 | Parameters                                         | Description                     | Example                            |
|-------------|----------------------------------------|----------------------------------------------------|---------------------------------|------------------------------------|
| **decimal** | `decimal($column, $precision, $scale)` | `precision`: total digits, `scale`: decimal places | Exact fixed-point number        | `$schema->decimal('price', 10, 2)` |
| **float**   | `float($column)`                       | None                                               | Single-precision floating-point | `$schema->float('rating')`         |
| **double**  | `double($column)`                      | None                                               | Double-precision floating-point | `$schema->double('coordinates')`   |

**Choosing the Right Type:**

```php
// Money/currency - use DECIMAL for exact values
$schema->decimal('price', 10, 2);        // Up to 99,999,999.99
$schema->decimal('tax_rate', 5, 4);      // Up to 9.9999 (e.g., 0.0825 = 8.25%)

// Scientific calculations - use FLOAT/DOUBLE
$schema->double('latitude');
$schema->double('longitude');
$schema->float('temperature_celsius');

// Avoid FLOAT/DOUBLE for money
// ❌ BAD: $schema->float('price');
// ✅ GOOD: $schema->decimal('price', 10, 2);
```

#### Date and Time Types

| Type          | Method                          | Parameters                            | Description                          | Example                              |
|---------------|---------------------------------|---------------------------------------|--------------------------------------|--------------------------------------|
| **datetime**  | `datetime($column, $precision)` | `precision`: fractional seconds (0-6) | Date and time with timezone handling | `$schema->datetime('created_at', 6)` |
| **date**      | `date($column)`                 | None                                  | Date only (no time component)        | `$schema->date('birth_date')`        |
| **time**      | `time($column)`                 | None                                  | Time only (no date component)        | `$schema->time('opening_time')`      |
| **timestamp** | `timestamp($column)`            | None                                  | Unix timestamp                       | `$schema->timestamp('updated_at')`   |

**DateTime Precision:**

```php
// No fractional seconds (most common)
$schema->datetime('created_at');

// With microseconds (for high-precision timestamps)
$schema->datetime('event_occurred_at', 6); // Stores up to microseconds

// Different databases handle precision differently:
// - MySQL: datetime(6) stores up to 6 decimal places
// - PostgreSQL: timestamp(6) with time zone
// - SQL Server: datetime2(6) instead of datetime
```

**Timezone Behavior:**

```php
// DBAL automatically handles UTC conversion for datetime columns
$schema->datetime('created_at')->defaultValue(AbstractColumn::DATETIME_NOW);

// Current timestamp as default
use Cycle\Database\Schema\AbstractColumn;
$schema->datetime('updated_at')->defaultValue(AbstractColumn::DATETIME_NOW);
```

> **Important**  
> DBAL automatically forces UTC timezone for `datetime` and `date` columns to ensure consistency across different server
> configurations.

#### Binary Types

| Type           | Method                | Max Size            | Description          | Example                        |
|----------------|-----------------------|---------------------|----------------------|--------------------------------|
| **binary**     | `binary($column)`     | 65,535 bytes        | Standard binary data | `$schema->binary('file_data')` |
| **tinyBinary** | `tinyBinary($column)` | 255 bytes           | Small binary field   | `$schema->tinyBinary('icon')`  |
| **longBinary** | `longBinary($column)` | 4,294,967,295 bytes | Large binary data    | `$schema->longBinary('video')` |

**Practical Use Cases:**

```php
// Small images or icons
$schema->binary('avatar');

// File uploads
$schema->longBinary('document')->nullable(true);

// Hash values
$schema->binary('password_hash');

// Consider storing files externally for very large data
```

#### Special Types

| Type          | Method               | Description                                              | Database Support                                 | Example                                |
|---------------|----------------------|----------------------------------------------------------|--------------------------------------------------|----------------------------------------|
| **boolean**   | `boolean($column)`   | True/false values (stored as 1/0 in most databases)      | All databases                                    | `$schema->boolean('is_active')`        |
| **json**      | `json($column)`      | JSON data structure                                      | Native in PostgreSQL, emulated as text in others | `$schema->json('metadata')`            |
| **uuid**      | `uuid($column)`      | UUID/GUID identifier                                     | Database-specific                                | `$schema->uuid('external_id')`         |
| **snowflake** | `snowflake($column)` | Twitter Snowflake ID                                     | Database-specific                                | `$schema->snowflake('distributed_id')` |
| **ulid**      | `ulid($column)`      | Universally Unique Lexicographically Sortable Identifier | Database-specific                                | `$schema->ulid('sortable_id')`         |

**Boolean Usage:**

```php
// Feature flags
$schema->boolean('is_active')->defaultValue(true);
$schema->boolean('email_verified')->defaultValue(false);

// Permissions
$schema->boolean('can_edit')->nullable(false);
```

**JSON Usage:**

```php
// Structured metadata
$schema->json('user_preferences');
$schema->json('api_response');

// Default JSON value
$schema->json('settings')->defaultValue([
    'theme' => 'light',
    'notifications' => true
]);
```

### Enum Types

Enums provide a way to restrict column values to a predefined set. MySQL supports enums natively; other databases
emulate them using string columns with CHECK constraints.

**Basic Enum Declaration:**

```php
// Define allowed values
$schema->column('status')->enum(['active', 'pending', 'disabled']);

// Using shortcut
$schema->enum('priority', ['low', 'medium', 'high', 'urgent']);
```

**With Default Values:**

```php
$schema->enum('status', ['active', 'pending', 'disabled'])
    ->defaultValue('pending');

$schema->enum('role', ['user', 'admin', 'moderator'])
    ->defaultValue('user')
    ->nullable(false);
```

**Modifying Enum Values:**

```php
// You can add or remove values by redefining the enum
$schema = $database->table('users')->getSchema();

// Add new status value
$schema->enum('status', ['active', 'pending', 'disabled', 'archived'])
    ->defaultValue('pending');

$schema->save(); // DBAL handles the modification
```

**Best Practices:**

```php
// ✅ GOOD: Use enums for fixed, small sets of values
$schema->enum('status', ['draft', 'published', 'archived']);
$schema->enum('gender', ['male', 'female', 'other']);

// ❌ AVOID: Large or frequently changing value sets
// Instead, use a lookup table with foreign key
```

### Column Attributes

#### Nullable Columns

By default, all columns are nullable. Use `nullable()` to change this:

```php
// Allow NULL values (default)
$schema->string('middle_name')->nullable(true);

// Require non-NULL values
$schema->string('email')->nullable(false);
$schema->integer('user_id')->nullable(false);

// When adding NOT NULL columns to existing tables with data
$schema->string('phone')
    ->nullable(false)
    ->defaultValue(''); // Provides value for existing rows
```

#### Default Values

Set default values for columns:

```php
// Scalar defaults
$schema->integer('views')->defaultValue(0);
$schema->boolean('is_active')->defaultValue(true);
$schema->string('status')->defaultValue('pending');

// NULL as default
$schema->string('optional_field')->defaultValue(null);

// Database functions (datetime)
use Cycle\Database\Schema\AbstractColumn;
$schema->datetime('created_at')->defaultValue(AbstractColumn::DATETIME_NOW);

// JSON defaults
$schema->json('settings')->defaultValue([
    'notifications' => true,
    'theme' => 'light'
]);
```

**Remove Default Value:**

```php
$schema->string('status')->defaultValue(null); // Remove default
```

#### Column Modification Chain

Combine multiple attributes:

```php
$schema->string('username', 64)
    ->nullable(false)
    ->unique();

$schema->decimal('balance', 10, 2)
    ->nullable(false)
    ->defaultValue(0.00);

$schema->datetime('created_at', 6)
    ->nullable(false)
    ->defaultValue(AbstractColumn::DATETIME_NOW);

$schema->enum('status', ['active', 'inactive'])
    ->nullable(false)
    ->defaultValue('active')
    ->index(); // Add simple index
```

## Primary Keys

### Auto-increment Primary Keys

The simplest approach uses `primary()` or `bigPrimary()`:

```php
$schema = $database->table('users')->getSchema();

// Standard auto-increment (32-bit)
$schema->primary('id');

// Or 64-bit for very large tables
$schema->bigPrimary('id');

$schema->save();
```

**Generated SQL (MySQL):**

```sql
CREATE TABLE users
(
    id INT (11) NOT NULL AUTO_INCREMENT,
    PRIMARY KEY (id)
) ENGINE=InnoDB;
```

### Composite Primary Keys

Create primary keys from multiple columns:

```php
$schema = $database->table('user_roles')->getSchema();

$schema->integer('user_id')->nullable(false);
$schema->integer('role_id')->nullable(false);

// Define composite primary key
$schema->setPrimaryKeys(['user_id', 'role_id']);

$schema->save();
```

**Generated SQL (MySQL):**

```sql
CREATE TABLE ` user_roles `
(
    `
    user_id
    `
    INT
(
    11
) NOT NULL,
    ` role_id ` INT
(
    11
) NOT NULL,
    PRIMARY KEY
(
    `
    user_id
    `,
    `
    role_id
    `
)
    ) ENGINE=InnoDB;
```

### Custom Primary Key (Non-Auto-Increment)

```php
$schema = $database->table('countries')->getSchema();

// String primary key
$schema->string('code', 2)->nullable(false);
$schema->string('name', 255);

$schema->setPrimaryKeys(['code']);

$schema->save();
```

> **Important**  
> Primary keys can only be set during table creation. You cannot change them after the table exists.

## Indexes

### Simple Indexes

Create indexes for query performance:

```php
$schema = $database->table('users')->getSchema();

$schema->primary('id');
$schema->string('email', 255);
$schema->string('username', 64);
$schema->datetime('created_at');

// Method 1: Using index() method
$schema->index(['email']);

// Method 2: Inline with column definition
$schema->column('username')->index();

$schema->save();
```

### Unique Indexes

Enforce uniqueness constraints:

```php
// Unique constraint on single column
$schema->string('email', 255)->unique();

// Or using index() method
$schema->string('username', 64);
$schema->index(['username'])->unique(true);

$schema->save();
```

### Composite Indexes

Index multiple columns together:

```php
$schema = $database->table('posts')->getSchema();

$schema->primary('id');
$schema->integer('user_id');
$schema->datetime('created_at');
$schema->string('status', 20);

// Composite index for common queries
$schema->index(['user_id', 'created_at']);
$schema->index(['status', 'created_at']);

// Composite unique index
$schema->string('slug', 255);
$schema->integer('category_id');
$schema->index(['slug', 'category_id'])->unique(true);

$schema->save();
```

### Index Column Sorting

Some databases support specifying sort order in indexes:

```php
// Ascending order (default)
$schema->index(['created_at ASC']);

// Descending order
$schema->index(['created_at DESC']);

// Mixed sorting in composite index
$schema->index(['status ASC', 'created_at DESC']);
```

> **Note**  
> Not all databases support column sorting in indexes. DBAL will throw a `DriverException` if the feature is not
> supported.

### Modifying Indexes

Change index properties:

```php
$schema = $database->table('users')->getSchema();

// Make existing index unique
$schema->index(['email'])->unique(true);

// Remove unique constraint
$schema->index(['email'])->unique(false);

$schema->save();
```

### Index Limitations

**Important Restrictions:**

```php
// ❌ CANNOT index text/binary columns
// $schema->text('description')->index(); // Will fail

// ✅ Use string for indexable text
$schema->string('title', 255)->index();

// ❌ CANNOT create unique index on table with duplicate data
// Must clean data first

// ✅ Consider index size limits (database-specific)
// MySQL: ~767 bytes for InnoDB
$schema->string('long_field', 1000)->index(); // May fail on some engines
```

## Foreign Keys

### Basic Foreign Key

Link tables together with referential integrity:

```php
// Parent table
$users = $database->table('users')->getSchema();
$users->primary('id');
$users->string('name', 64);
$users->save();

// Child table with foreign key
$posts = $database->table('posts')->getSchema();
$posts->primary('id');
$posts->string('title', 255);
$posts->integer('user_id');

// Create foreign key
$posts->foreign('user_id')->references('users', 'id');

$posts->save();
```

**Generated SQL (MySQL):**

```sql
ALTER TABLE posts
    ADD CONSTRAINT posts_foreign_user_id_5f2a8b9c
        FOREIGN KEY (user_id)
            REFERENCES users (id)
            ON DELETE NO ACTION
            ON UPDATE NO ACTION;
```

### Foreign Key Actions

Control behavior when referenced records change:

```php
$posts = $database->table('posts')->getSchema();
$posts->integer('user_id');

$foreignKey = $posts->foreign('user_id')->references('users', 'id');

// Delete posts when user is deleted
$foreignKey->onDelete(ForeignKeyInterface::CASCADE);

// Update foreign key when parent key changes
$foreignKey->onUpdate(ForeignKeyInterface::CASCADE);

$posts->save();
```

**Available Actions:**

| Action        | Constant                         | Description                                              |
|---------------|----------------------------------|----------------------------------------------------------|
| **NO ACTION** | `ForeignKeyInterface::NO_ACTION` | Prevent deletion/update if child records exist (default) |
| **CASCADE**   | `ForeignKeyInterface::CASCADE`   | Delete/update child records automatically                |
| **SET NULL**  | `ForeignKeyInterface::SET_NULL`  | Set foreign key to NULL (requires nullable column)       |
| **RESTRICT**  | `ForeignKeyInterface::RESTRICT`  | Similar to NO ACTION, checked immediately                |

**Complete Example:**

```php
use Cycle\Database\ForeignKeyInterface;

// Posts are deleted when user is deleted
$posts->integer('author_id')->nullable(false);
$posts->foreign('author_id')
    ->references('users', 'id')
    ->onDelete(ForeignKeyInterface::CASCADE)
    ->onUpdate(ForeignKeyInterface::CASCADE);

// Comments orphaned when post is deleted
$comments->integer('post_id')->nullable(true);
$comments->foreign('post_id')
    ->references('posts', 'id')
    ->onDelete(ForeignKeyInterface::SET_NULL)
    ->onUpdate(ForeignKeyInterface::CASCADE);

// Orders cannot be deleted if they have items
$orderItems->integer('order_id')->nullable(false);
$orderItems->foreign('order_id')
    ->references('orders', 'id')
    ->onDelete(ForeignKeyInterface::RESTRICT)
    ->onUpdate(ForeignKeyInterface::CASCADE);
```

### Composite Foreign Keys

Reference multiple columns:

```php
// Parent table with composite primary key
$userProducts = $database->table('user_products')->getSchema();
$userProducts->integer('user_id')->nullable(false);
$userProducts->integer('product_id')->nullable(false);
$userProducts->setPrimaryKeys(['user_id', 'product_id']);
$userProducts->save();

// Child table referencing composite key
$favorites = $database->table('favorites')->getSchema();
$favorites->primary('id');
$favorites->integer('user_id');
$favorites->integer('product_id');

$favorites->foreign(['user_id', 'product_id'])
    ->references('user_products', ['user_id', 'product_id'])
    ->onDelete(ForeignKeyInterface::CASCADE);

$favorites->save();
```

### Disabling Automatic Index Creation

By default, foreign keys create an index. Disable if needed:

```php
$posts->integer('user_id');
$posts->foreign('user_id', false) // false = don't create index
    ->references('users', 'id');
```

### Database-Specific Considerations

```php
// Some databases (SQL Server) restrict CASCADE actions
// to avoid circular references. Plan your schema accordingly.

// Example: Cannot have mutual CASCADE relationships
// Table A -> CASCADE -> Table B
// Table B -> CASCADE -> Table A  // May be rejected
```

## Schema Modification

### Renaming Elements

#### Rename Columns

```php
$schema = $database->table('users')->getSchema();

// Method 1: Using setName()
$schema->column('email')->setName('email_address');

// Method 2: Using renameColumn()
$schema->renameColumn('old_name', 'new_name');

$schema->save();
```

**Generated SQL (MySQL):**

```sql
ALTER TABLE users
    CHANGE email email_address VARCHAR (255) NULL;
```

#### Rename Table

```php
$schema = $database->table('old_users')->getSchema();
$schema->setName('users');
$schema->save();
```

**Generated SQL:**

```sql
ALTER TABLE old_users
    RENAME TO users;
```

#### Rename Indexes

```php
$schema = $database->table('users')->getSchema();

$schema->renameIndex(
    ['email'],        // Column(s) that form the index
    'idx_user_email'  // New index name
);

$schema->save();
```

### Dropping Elements

#### Drop Columns

```php
$schema = $database->table('users')->getSchema();

$schema->dropColumn('middle_name');
$schema->dropColumn('legacy_field');

$schema->save();
```

**Generated SQL:**

```sql
ALTER TABLE users
    DROP COLUMN middle_name;
ALTER TABLE users
    DROP COLUMN legacy_field;
```

#### Drop Indexes

```php
$schema = $database->table('posts')->getSchema();

// Drop index by column(s)
$schema->dropIndex(['title']);
$schema->dropIndex(['user_id', 'created_at']); // Composite index

$schema->save();
```

#### Drop Foreign Keys

```php
$schema = $database->table('posts')->getSchema();

// Drop by column(s)
$schema->dropForeignKey(['user_id']);

$schema->save();
```

**Important Order:**

```php
// Always drop foreign keys before dropping referenced columns
$schema = $database->table('posts')->getSchema();

// 1. Drop foreign key first
$schema->dropForeignKey(['user_id']);

// 2. Then drop the column
$schema->dropColumn('user_id');

$schema->save();
```

#### Drop Table

```php
$schema = $database->table('old_table')->getSchema();

$schema->declareDropped();
$schema->save();
```

## Advanced Operations

### Clean Table Schema

Reset table schema to only contain declared elements:

```php
$schema = $database->table('users')->getSchema();

// Clear current schema state
$schema->setState(null);

// Now redefine only the columns you want
$schema->primary('id');
$schema->string('email', 255);
$schema->string('username', 64);

// All other columns will be dropped
$schema->save();
```

**Use Cases:**

- Removing legacy columns not in your application model
- Cleaning up after major schema refactoring
- Enforcing strict schema definitions

> **Warning**  
> This will drop any columns not declared in your schema. Use with caution on production databases.

### Working with Comparator

Inspect schema changes before applying them:

```php
$schema = $database->table('users')->getSchema();

// Make some changes
$schema->string('new_field');
$schema->dropColumn('old_field');
$schema->index(['email'])->unique(true);

// Get comparator to inspect changes
$comparator = $schema->getComparator();

// Check what will be added
$addedColumns = $comparator->addedColumns();
foreach ($addedColumns as $column) {
    echo "Will add column: " . $column->getName() . "\n";
}

// Check what will be dropped
$droppedColumns = $comparator->droppedColumns();
foreach ($droppedColumns as $column) {
    echo "Will drop column: " . $column->getName() . "\n";
}

// Check what will be modified
$alteredColumns = $comparator->alteredColumns();
foreach ($alteredColumns as [$new, $old]) {
    echo "Will modify column: " . $new->getName() . "\n";
}

// Check if there are any changes
if ($comparator->hasChanges()) {
    echo "Schema has pending changes\n";
    
    // Optionally save
    $schema->save();
} else {
    echo "Schema is up to date\n";
}
```

**Comparator Methods:**

| Method                 | Returns                | Description                                    |
|------------------------|------------------------|------------------------------------------------|
| `hasChanges()`         | `bool`                 | Whether any changes exist                      |
| `isRenamed()`          | `bool`                 | Whether table is renamed                       |
| `isPrimaryChanged()`   | `bool`                 | Whether primary key changed                    |
| `addedColumns()`       | `AbstractColumn[]`     | New columns to create                          |
| `droppedColumns()`     | `AbstractColumn[]`     | Columns to remove                              |
| `alteredColumns()`     | `array`                | Modified columns (returns pairs of [new, old]) |
| `addedIndexes()`       | `AbstractIndex[]`      | New indexes to create                          |
| `droppedIndexes()`     | `AbstractIndex[]`      | Indexes to remove                              |
| `alteredIndexes()`     | `array`                | Modified indexes                               |
| `addedForeignKeys()`   | `AbstractForeignKey[]` | New foreign keys to create                     |
| `droppedForeignKeys()` | `AbstractForeignKey[]` | Foreign keys to remove                         |
| `alteredForeignKeys()` | `array`                | Modified foreign keys                          |

**Generate Migration Code:**

```php
$comparator = $schema->getComparator();

// Use comparator data to generate migration files
foreach ($comparator->addedColumns() as $column) {
    $migrationCode = sprintf(
        '$schema->%s(\'%s\');',
        $column->getAbstractType(),
        $column->getName()
    );
    // Write to migration file...
}
```

### Syncing Multiple Tables

When creating multiple related tables, use `Reflector` to handle dependencies automatically:

```php
use Cycle\Database\Schema\Reflector;

// Define first table
$users = $database->table('users')->getSchema();
$users->primary('id');
$users->string('username', 64);

// Define related table
$posts = $database->table('posts')->getSchema();
$posts->primary('id');
$posts->string('title', 255);
$posts->integer('user_id');
$posts->foreign('user_id')->references('users', 'id');

// Define another related table
$comments = $database->table('comments')->getSchema();
$comments->primary('id');
$comments->text('content');
$comments->integer('post_id');
$comments->foreign('post_id')->references('posts', 'id');

// Create reflector and add tables
$reflector = new Reflector();
$reflector->addTable($users);
$reflector->addTable($posts);
$reflector->addTable($comments);

// Reflector automatically orders tables by dependencies
// and syncs them in the correct order
$reflector->run();
```

**How It Works:**

1. **Dependency Analysis**: Reflector analyzes foreign key relationships
2. **Topological Sort**: Tables are sorted to respect dependencies
3. **Transactional Execution**: All changes execute in a transaction
4. **Proper Order**: Parent tables are created before child tables

**Manual Ordering (if needed):**

```php
$reflector = new Reflector();
$reflector->addTable($users);
$reflector->addTable($posts);
$reflector->addTable($comments);

// Get tables in dependency order
$sortedTables = $reflector->sortedTables();

foreach ($sortedTables as $table) {
    echo "Processing: " . $table->getName() . "\n";
}
```

**Benefits of Using Reflector:**

- Handles circular dependencies
- Manages foreign key constraints properly
- Executes all operations transactionally
- Prevents constraint violation errors
- Works across multiple databases

**Example with Multiple Databases:**

```php
$dbPrimary = $dbal->database('primary');
$dbSecondary = $dbal->database('secondary');

$reflector = new Reflector();

// Add tables from different databases
$reflector->addTable($dbPrimary->table('users')->getSchema());
$reflector->addTable($dbPrimary->table('posts')->getSchema());
$reflector->addTable($dbSecondary->table('logs')->getSchema());

// Reflector handles multiple database transactions
$reflector->run();
```

## Best Practices

### 1. Use Appropriate Column Types

Choose the smallest type that fits your data:

```php
// ✅ GOOD: Right-sized types
$schema->tinyInteger('age');           // 0-255
$schema->string('country_code', 2);    // 'US', 'UK'
$schema->decimal('price', 10, 2);      // Exact money values

// ❌ AVOID: Oversized types
$schema->bigInteger('age');            // Wastes space
$schema->string('country_code', 255);  // Wastes space
$schema->float('price');               // Imprecise for money
```

### 2. Index Strategic Columns

```php
// ✅ GOOD: Index frequently queried columns
$schema->string('email', 255)->unique();
$schema->index(['user_id', 'created_at']); // For sorting
$schema->index(['status', 'priority']);    // For filtering

// ❌ AVOID: Over-indexing
// Don't index columns rarely used in queries
// Don't index large text fields
```

### 3. Set Appropriate Defaults and Nullability

```php
// ✅ GOOD: Clear defaults and null handling
$schema->boolean('is_active')->defaultValue(true)->nullable(false);
$schema->integer('view_count')->defaultValue(0)->nullable(false);
$schema->datetime('created_at')->defaultValue(AbstractColumn::DATETIME_NOW);

// ❌ AVOID: Ambiguous nullable booleans
$schema->boolean('is_verified')->nullable(true); // NULL vs false?
```

### 4. Use Foreign Keys for Data Integrity

```php
// ✅ GOOD: Enforce referential integrity
$schema->foreign('user_id')
    ->references('users', 'id')
    ->onDelete(ForeignKeyInterface::CASCADE);

// ❌ AVOID: Orphaned records
// Without FK, you may have posts pointing to non-existent users
```

### 5. Plan Composite Indexes Carefully

```php
// ✅ GOOD: Most selective column first
$schema->index(['status', 'created_at', 'user_id']);
// Useful for: WHERE status = ? ORDER BY created_at

// ❌ AVOID: Poor ordering
$schema->index(['created_at', 'status']);
// Can't efficiently filter by status alone
```

### 6. Use Transactions for Related Changes

```php
// ✅ GOOD: Multiple related schemas
$reflector = new Reflector();
$reflector->addTable($users);
$reflector->addTable($posts);
$reflector->run(); // All or nothing

// ❌ AVOID: Individual saves
$users->save();
$posts->save(); // May fail if users didn't save
```

### 7. Review Changes Before Production

```php
// ✅ GOOD: Check before saving
$comparator = $schema->getComparator();
if ($comparator->droppedColumns()) {
    // Confirm you want to drop these columns
    foreach ($comparator->droppedColumns() as $col) {
        echo "WARNING: Will drop " . $col->getName() . "\n";
    }
}
$schema->save();
```

### 8. Consider Migration Systems

For production applications, use migrations instead of direct schema saves:

```php
// ✅ GOOD: Generate migrations for review
// See the Migrations documentation for details

// ❌ AVOID: Direct schema saves in production
// $schema->save(); // Skip in production, use migrations
```

### 9. Document Complex Schemas

```php
// ✅ GOOD: Add comments for clarity
$schema->string('status', 20)
    ->enum(['draft', 'review', 'published', 'archived'])
    ->defaultValue('draft');
// Status workflow: draft -> review -> published -> archived

$schema->decimal('tax_rate', 5, 4);
// Example: 0.0825 for 8.25% tax rate
```

### 10. Test Schema Changes

```php
// ✅ GOOD: Test on development/staging first
// 1. Test schema changes locally
// 2. Verify in staging environment
// 3. Back up production before applying
// 4. Have rollback plan ready

// ❌ AVOID: Untested production changes
```

## Type Compatibility Notes

When altering column types, be aware of database-specific conversion limitations:

**Safe Conversions:**

- `integer` → `bigInteger`
- `string(64)` → `string(128)` (increasing length)
- `nullable(true)` → `nullable(false)` (with default or data cleanup)

**Potentially Unsafe:**

- `string` → `integer` (data loss if non-numeric values exist)
- `bigInteger` → `integer` (data loss if values exceed range)
- `string(128)` → `string(64)` (data truncation)

**Database-Specific:**

- MySQL: More flexible type conversions
- PostgreSQL: Stricter type checking
- SQLite: Very flexible (dynamic typing)
- SQL Server: Moderate strictness

Always test schema changes in a non-production environment first.