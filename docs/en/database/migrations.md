# Database - Migrations

Migrations provide a structured and organized way to alter your database schema over time. This package adds versioning
capabilities for your database schema and simplifies deployment of schema changes across different environments.

## Installation

Install the migrations package via Composer:

```bash
composer require cycle/migrations
```

## Configuration

### Migration Config

The `MigrationConfig` class allows you to customize migration behavior and storage settings:

```php
use Cycle\Migrations\Config\MigrationConfig;

$config = new MigrationConfig([
    // Directory where migration files are stored
    'directory' => __DIR__ . '/../migrations/',
    
    // Additional vendor directories for third-party migrations
    'vendorDirectories' => [
        __DIR__ . '/../vendor/some-package/migrations/',
    ],
    
    // Database table name for tracking migration status
    'table' => 'migrations',
    
    // Namespace for generated migration classes
    'namespace' => 'Migration',
    
    // When true, migrations run without user confirmation (use carefully in production)
    'safe' => false
]);
```

**Configuration Options:**

| Option              | Type     | Default        | Description                                       |
|---------------------|----------|----------------|---------------------------------------------------|
| `directory`         | `string` | `''`           | Primary directory for migration files             |
| `vendorDirectories` | `array`  | `[]`           | Additional directories for vendor migrations      |
| `table`             | `string` | `'migrations'` | Table name for migration tracking                 |
| `namespace`         | `string` | `'Migration'`  | Namespace for generated migration classes         |
| `safe`              | `bool`   | `false`        | Skip confirmation prompts when running migrations |

### Initializing Migrator

Create and configure the migrator instance:

```php
use Cycle\Migrations\Migrator;
use Cycle\Migrations\FileRepository;

$migrator = new Migrator(
    $config,
    $dbal, // DatabaseProviderInterface instance
    new FileRepository($config)
);

// Initialize the migration tracking table
$migrator->configure();
```

The `configure()` method creates the migration tracking table if it doesn't exist and ensures its structure is up to
date.

## Creating Migrations

### Manual Migration Creation

Migrations are PHP classes that extend the `Cycle\Migrations\Migration` base class. Each migration must implement two
methods: `up()` for applying changes and `down()` for reverting them.

**Basic Migration Structure:**

```php
<?php

declare(strict_types=1);

namespace Migration;

use Cycle\Migrations\Migration;

class CreateUsersTableMigration extends Migration
{
    protected const DATABASE = null; // Use default database, or specify: 'default', 'secondary', etc.

    /**
     * Apply migration changes
     */
    public function up(): void
    {
        $this->table('users')
            ->addColumn('id', 'primary')
            ->addColumn('email', 'string', ['length' => 255, 'nullable' => false])
            ->addColumn('username', 'string', ['length' => 64, 'nullable' => false])
            ->addColumn('password_hash', 'string', ['length' => 255])
            ->addColumn('created_at', 'datetime')
            ->addColumn('updated_at', 'datetime', ['nullable' => true])
            ->addIndex(['email'], ['unique' => true])
            ->addIndex(['username'], ['unique' => true])
            ->create();
    }

    /**
     * Revert migration changes
     */
    public function down(): void
    {
        $this->table('users')->drop();
    }
}
```

**Migration File Naming:**

Migration files follow the format: `YYYYMMDD.HHmmss_ChunkID_migration_name.php`

- **Timestamp**: `YYYYMMDD.HHmmss` format for ordering
- **ChunkID**: Numeric identifier for migrations created at the same time
- **Name**: Descriptive name using snake_case

Example: `20240115.143022_0_create_users_table.php`

### Automatic Migration Generation

Cycle ORM can automatically generate migrations by comparing your entity definitions with the current database schema.

**Installation:**

```bash
composer require cycle/schema-migrations-generator
```

**Usage:**

```php
use Cycle\Schema\Registry;
use Cycle\Schema\Generator\Migrations\GenerateMigrations;

$registry = new Registry($dbal);
// Register entities...

$generator = new GenerateMigrations(
    $migrator->getRepository(),
    $migrator->getConfig()
);

// Generate migrations for schema changes
$generator->run($registry);
```

The generator analyzes differences between your entity schema and the database, creating migrations to synchronize them.
Each database receives its own migration file.

### Migration Strategies

The migration generator supports different strategies for organizing migration files:

#### Single File Strategy (Default)

Generates one migration file per database, grouping all changes together.

```php
use Cycle\Schema\Generator\Migrations\GenerateMigrations;
use Cycle\Schema\Generator\Migrations\Strategy\SingleFileStrategy;

$generator = new GenerateMigrations(
    $migrator->getRepository(),
    $migrator->getConfig(),
    new SingleFileStrategy($migrator->getConfig())
);
```

**Benefits:**

- Fewer migration files
- Simpler migration history
- Atomic changes per database

#### Multiple Files Strategy

Generates separate migration files for each table, improving organization and readability.

```php
use Cycle\Schema\Generator\Migrations\GenerateMigrations;
use Cycle\Schema\Generator\Migrations\Strategy\MultipleFilesStrategy;
use Cycle\Schema\Generator\Migrations\NameBasedOnChangesGenerator;

$generator = new GenerateMigrations(
    $migrator->getRepository(),
    $migrator->getConfig(),
    new MultipleFilesStrategy(
        $migrator->getConfig(),
        new NameBasedOnChangesGenerator()
    )
);
```

**Benefits:**

- Better organization for large schemas
- Easier to review individual table changes
- More granular version control
- Parallel development friendly

**Custom Name Generator:**

The `NameBasedOnChangesGenerator` creates descriptive migration names based on the operations performed. You can
implement `NameGeneratorInterface` for custom naming logic:

```php
use Cycle\Schema\Generator\Migrations\NameGeneratorInterface;

class CustomNameGenerator implements NameGeneratorInterface
{
    public function generate(array $changes): string
    {
        // Your custom naming logic
        return 'custom_migration_name';
    }
}
```

## Migration Operations

The migration system provides a fluent API through the `TableBlueprint` class for defining schema changes. All
operations are chainable and must conclude with a finalization method.

### Table Operations

#### Creating Tables

Use the `create()` method to finalize table creation:

```php
public function up(): void
{
    $this->table('products')
        ->addColumn('id', 'primary')
        ->addColumn('name', 'string', ['length' => 255])
        ->addColumn('price', 'decimal', ['precision' => 10, 'scale' => 2])
        ->addColumn('stock', 'integer', ['default' => 0])
        ->create();
}
```

**Important:** At least one column must be defined before calling `create()`.

#### Updating Tables

Use the `update()` method to modify existing table structure:

```php
public function up(): void
{
    $this->table('products')
        ->addColumn('description', 'text', ['nullable' => true])
        ->addColumn('category_id', 'integer')
        ->addIndex(['category_id'])
        ->update();
}
```

#### Dropping Tables

Use the `drop()` method to remove a table:

```php
public function down(): void
{
    $this->table('products')->drop();
}
```

#### Renaming Tables

Use the `rename()` method to change a table's name:

```php
public function up(): void
{
    $this->table('old_products')->rename('products');
}
```

#### Setting Primary Keys

Define primary keys during table creation:

```php
public function up(): void
{
    $this->table('composite_example')
        ->addColumn('user_id', 'integer')
        ->addColumn('resource_id', 'integer')
        ->addColumn('permission', 'string')
        ->setPrimaryKeys(['user_id', 'resource_id']) // Composite primary key
        ->create();
}
```

> **Note:** `setPrimaryKeys()` can only be used during table creation, not when updating.

### Column Operations

#### Adding Columns

The `addColumn()` method adds new columns to a table:

```php
public function up(): void
{
    $this->table('users')
        ->addColumn('email', 'string', [
            'length' => 255,
            'nullable' => false,
            'default' => null
        ])
        ->update();
}
```

**Common Column Options:**

| Option         | Type    | Description                          |
|----------------|---------|--------------------------------------|
| `nullable`     | `bool`  | Allow NULL values (default: `false`) |
| `default`      | `mixed` | Default value for the column         |
| `defaultValue` | `mixed` | Alias for `default`                  |
| `length`       | `int`   | Maximum length for string types      |
| `size`         | `int`   | Alias for `length`                   |
| `precision`    | `int`   | Total digits for numeric types       |
| `scale`        | `int`   | Decimal places for numeric types     |
| `values`       | `array` | Allowed values for enum types        |

**Column Type Examples:**

```php
// String types
->addColumn('username', 'string', ['length' => 64])
->addColumn('bio', 'text')

// Numeric types
->addColumn('age', 'integer')
->addColumn('price', 'decimal', ['precision' => 10, 'scale' => 2])
->addColumn('rating', 'float')

// Date/Time types
->addColumn('created_at', 'datetime')
->addColumn('published_date', 'date')
->addColumn('start_time', 'time')
->addColumn('updated_at', 'timestamp')

// Binary and JSON
->addColumn('avatar', 'binary')
->addColumn('metadata', 'json')

// Boolean
->addColumn('is_active', 'boolean', ['default' => true])

// Enum
->addColumn('status', 'enum', ['values' => ['pending', 'active', 'inactive']])
```

> Read more about available column types in the [Schema Declaration](/docs/en/database/declaration.md) documentation.

#### Altering Columns

The `alterColumn()` method modifies existing column definitions:

```php
public function up(): void
{
    $this->table('users')
        ->alterColumn('username', 'string', [
            'length' => 128, // Changed from 64
            'nullable' => false
        ])
        ->update();
}
```

**Example - Making a Column Nullable:**

```php
public function up(): void
{
    $this->table('posts')
        ->alterColumn('excerpt', 'text', ['nullable' => true])
        ->update();
}

public function down(): void
{
    $this->table('posts')
        ->alterColumn('excerpt', 'text', ['nullable' => false])
        ->update();
}
```

#### Renaming Columns

The `renameColumn()` method changes a column's name:

```php
public function up(): void
{
    $this->table('users')
        ->renameColumn('full_name', 'display_name')
        ->update();
}

public function down(): void
{
    $this->table('users')
        ->renameColumn('display_name', 'full_name')
        ->update();
}
```

#### Dropping Columns

The `dropColumn()` method removes columns:

```php
public function up(): void
{
    $this->table('users')
        ->dropColumn('legacy_field')
        ->update();
}

public function down(): void
{
    $this->table('users')
        ->addColumn('legacy_field', 'string', ['nullable' => true])
        ->update();
}
```

**Dropping Multiple Columns:**

```php
$this->table('users')
    ->dropColumn('temp_field_1')
    ->dropColumn('temp_field_2')
    ->dropColumn('temp_field_3')
    ->update();
```

### Index Operations

#### Adding Indexes

The `addIndex()` method creates indexes on table columns:

```php
public function up(): void
{
    $this->table('users')
        ->addIndex(['email'], ['unique' => true])
        ->addIndex(['created_at'])
        ->addIndex(['last_name', 'first_name']) // Composite index
        ->update();
}
```

**Index Options:**

| Option   | Type     | Default        | Description              |
|----------|----------|----------------|--------------------------|
| `unique` | `bool`   | `false`        | Create unique constraint |
| `name`   | `string` | Auto-generated | Custom index name        |

**Named Index Example:**

```php
$this->table('products')
    ->addIndex(['category_id', 'price'], [
        'name' => 'idx_products_category_price',
        'unique' => false
    ])
    ->update();
```

#### Altering Indexes

The `alterIndex()` method modifies existing indexes:

```php
public function up(): void
{
    $this->table('users')
        ->alterIndex(['email'], ['unique' => false]) // Remove unique constraint
        ->update();
}
```

#### Dropping Indexes

The `dropIndex()` method removes indexes:

```php
public function up(): void
{
    $this->table('users')
        ->dropIndex(['email'])
        ->update();
}

public function down(): void
{
    $this->table('users')
        ->addIndex(['email'], ['unique' => true])
        ->update();
}
```

### Foreign Key Operations

#### Adding Foreign Keys

The `addForeignKey()` method creates foreign key constraints:

```php
public function up(): void
{
    $this->table('posts')
        ->addColumn('author_id', 'integer')
        ->addForeignKey(
            ['author_id'],           // Local columns
            'users',                 // Foreign table
            ['id'],                  // Foreign columns
            [
                'delete' => 'CASCADE',
                'update' => 'CASCADE',
                'indexCreate' => true
            ]
        )
        ->update();
}
```

**Foreign Key Options:**

| Option        | Type     | Default        | Description                                                      |
|---------------|----------|----------------|------------------------------------------------------------------|
| `delete`      | `string` | `'NO_ACTION'`  | Action on delete: `CASCADE`, `SET_NULL`, `RESTRICT`, `NO_ACTION` |
| `update`      | `string` | `'NO_ACTION'`  | Action on update: `CASCADE`, `SET_NULL`, `RESTRICT`, `NO_ACTION` |
| `indexCreate` | `bool`   | `true`         | Automatically create index on foreign key columns                |
| `name`        | `string` | Auto-generated | Custom constraint name                                           |

**Referential Actions:**

- `CASCADE`: Delete/update related records
- `SET_NULL`: Set foreign key to NULL (requires nullable column)
- `RESTRICT`: Prevent delete/update if related records exist
- `NO_ACTION`: Similar to RESTRICT, but checked after other constraints

**Composite Foreign Key Example:**

```php
$this->table('order_items')
    ->addForeignKey(
        ['user_id', 'product_id'],
        'user_products',
        ['user_id', 'product_id'],
        ['delete' => 'CASCADE']
    )
    ->update();
```

#### Altering Foreign Keys

The `alterForeignKey()` method modifies foreign key constraints:

```php
public function up(): void
{
    $this->table('posts')
        ->alterForeignKey(
            ['author_id'],
            'users',
            ['id'],
            ['delete' => 'SET_NULL'] // Changed from CASCADE
        )
        ->update();
}
```

#### Dropping Foreign Keys

The `dropForeignKey()` method removes foreign key constraints:

```php
public function up(): void
{
    $this->table('posts')
        ->dropForeignKey(['author_id'])
        ->update();
}

public function down(): void
{
    $this->table('posts')
        ->addForeignKey(
            ['author_id'],
            'users',
            ['id'],
            ['delete' => 'CASCADE']
        )
        ->update();
}
```

**Important:** Always drop foreign keys before dropping columns they reference.

## Running Migrations

### Executing Migrations

The `run()` method executes pending migrations in chronological order:

```php
use Cycle\Migrations\Capsule;

// Run next pending migration
$migration = $migrator->run(new Capsule($dbal->database()));

if ($migration !== null) {
    echo "Executed migration: " . $migration->getState()->getName();
} else {
    echo "No pending migrations";
}
```

**Running All Pending Migrations:**

```php
while (($migration = $migrator->run()) !== null) {
    $state = $migration->getState();
    echo sprintf(
        "Migration '%s' executed at %s\n",
        $state->getName(),
        $state->getTimeExecuted()->format('Y-m-d H:i:s')
    );
}
```

**Checking Migration Status:**

```php
// Check if migrations are configured
if (!$migrator->isConfigured()) {
    $migrator->configure();
}

// Get all migrations with their status
foreach ($migrator->getMigrations() as $migration) {
    $state = $migration->getState();
    
    echo sprintf(
        "%s: %s (created: %s)\n",
        $state->getName(),
        $state->getStatus() === State::STATUS_EXECUTED ? 'Executed' : 'Pending',
        $state->getTimeCreated()->format('Y-m-d H:i:s')
    );
}
```

**Migration State:**

The `State` class tracks migration status:

```php
use Cycle\Migrations\State;

$state = $migration->getState();

// Migration status constants
State::STATUS_UNDEFINED  // -1: Unknown status
State::STATUS_PENDING    //  0: Not executed
State::STATUS_EXECUTED   //  1: Already executed

// State methods
$state->getName();           // Migration name
$state->getStatus();         // Current status (int)
$state->getTimeCreated();    // DateTimeInterface when migration was created
$state->getTimeExecuted();   // DateTimeInterface when executed (null if pending)
```

**Transaction Handling:**

Migrations automatically run within database transactions. If any operation fails, the entire migration is rolled back:

```php
public function up(): void
{
    // All these operations execute in a single transaction
    $this->table('users')
        ->addColumn('verified', 'boolean', ['default' => false])
        ->update();
    
    // If this fails, the column addition is rolled back
    $this->database()->insert('users')->values([
        'verified' => true
    ])->run();
}
```

### Rolling Back Migrations

The `rollback()` method reverts the last executed migration:

```php
// Rollback last migration
$migration = $migrator->rollback(new Capsule($dbal->database()));

if ($migration !== null) {
    echo "Rolled back migration: " . $migration->getState()->getName();
} else {
    echo "No migrations to rollback";
}
```

**Rolling Back Multiple Migrations:**

```php
// Rollback last 3 migrations
for ($i = 0; $i < 3; $i++) {
    $migration = $migrator->rollback();
    if ($migration === null) {
        break;
    }
    echo "Rolled back: " . $migration->getState()->getName() . "\n";
}
```

**Best Practices for Rollbacks:**

1. **Always implement the `down()` method** - Never leave it empty
2. **Test rollbacks** - Ensure they properly revert changes
3. **Handle data loss** - Dropping columns loses data permanently
4. **Order matters** - Rollback operations should reverse `up()` operations

**Example - Rollback with Data Preservation:**

```php
public function up(): void
{
    // Add new required column with default value
    $this->table('users')
        ->addColumn('role', 'string', ['default' => 'user'])
        ->update();
}

public function down(): void
{
    // Rollback: drop the column (data will be lost)
    $this->table('users')
        ->dropColumn('role')
        ->update();
}
```

## Migration Blueprint API Reference

The `TableBlueprint` class provides the complete migration API. All methods are chainable and must conclude with a
finalization method.

### Table Finalization Methods

These methods execute the migration operations:

| Method                 | Description                           | Usage                          |
|------------------------|---------------------------------------|--------------------------------|
| `create()`             | Create new table with defined columns | Must have at least one column  |
| `update()`             | Apply changes to existing table       | Use for ALTER TABLE operations |
| `drop()`               | Remove table from database            | Irreversible operation         |
| `rename(string $name)` | Change table name                     | Specify new table name         |

### Column Methods

| Method           | Parameters                                        | Description              |
|------------------|---------------------------------------------------|--------------------------|
| `addColumn()`    | `string $name, string $type, array $options = []` | Add new column to table  |
| `alterColumn()`  | `string $name, string $type, array $options = []` | Modify existing column   |
| `renameColumn()` | `string $oldName, string $newName`                | Rename column            |
| `dropColumn()`   | `string $name`                                    | Remove column from table |

### Index Methods

| Method         | Parameters                            | Description             |
|----------------|---------------------------------------|-------------------------|
| `addIndex()`   | `array $columns, array $options = []` | Create index on columns |
| `alterIndex()` | `array $columns, array $options`      | Modify existing index   |
| `dropIndex()`  | `array $columns`                      | Remove index from table |

### Foreign Key Methods

| Method              | Parameters                                                                      | Description                   |
|---------------------|---------------------------------------------------------------------------------|-------------------------------|
| `addForeignKey()`   | `array $columns, string $foreignTable, array $foreignKeys, array $options = []` | Create foreign key constraint |
| `alterForeignKey()` | `array $columns, string $foreignTable, array $foreignKeys, array $options = []` | Modify foreign key constraint |
| `dropForeignKey()`  | `array $columns`                                                                | Remove foreign key constraint |

### Primary Key Methods

| Method             | Parameters       | Description                                        |
|--------------------|------------------|----------------------------------------------------|
| `setPrimaryKeys()` | `array $columns` | Define primary key(s) - only during table creation |

### Accessing Database

Inside migration methods, you can access the database instance:

```php
public function up(): void
{
    // Access database for data operations
    $db = $this->database();
    
    // Insert initial data
    $db->insert('roles')->values([
        ['name' => 'admin'],
        ['name' => 'user'],
    ])->run();
}
```

### Accessing Table Schema

Get the underlying table schema for advanced operations:

```php
public function up(): void
{
    $blueprint = $this->table('users');
    $schema = $blueprint->getSchema();
    
    // Direct schema manipulation if needed
    // (prefer using blueprint methods when possible)
}
```

## Best Practices

### 1. Always Write Reversible Migrations

Every migration should properly implement both `up()` and `down()` methods:

```php
// Good - Fully reversible
public function up(): void
{
    $this->table('users')->addColumn('deleted_at', 'datetime', ['nullable' => true])->update();
}

public function down(): void
{
    $this->table('users')->dropColumn('deleted_at')->update();
}
```

### 2. Use Descriptive Migration Names

Choose names that clearly describe the change:

```php
// Good naming
CreateUsersTableMigration
AddEmailIndexToUsersTable
AlterPostsTableAddPublishedDate
RemoveLegacyColumnsFromProducts

// Poor naming
Migration1
UpdateTable
ChangeStuff
```

### 3. Keep Migrations Focused

Each migration should accomplish one logical change:

```php
// Good - Single responsibility
class AddUserEmailVerificationMigration extends Migration
{
    public function up(): void
    {
        $this->table('users')
            ->addColumn('email_verified_at', 'datetime', ['nullable' => true])
            ->addColumn('verification_token', 'string', ['length' => 64, 'nullable' => true])
            ->update();
    }
}

// Avoid - Multiple unrelated changes
class UpdateMultipleTablesMigration extends Migration
{
    public function up(): void
    {
        $this->table('users')->addColumn('field1', 'string')->update();
        $this->table('posts')->dropColumn('field2')->update();
        $this->table('comments')->renameColumn('old', 'new')->update();
    }
}
```

### 4. Handle Foreign Keys Carefully

Always drop foreign keys before dropping referenced columns or tables:

```php
public function down(): void
{
    // Correct order
    $this->table('posts')
        ->dropForeignKey(['author_id'])  // Drop FK first
        ->dropColumn('author_id')         // Then drop column
        ->update();
}
```

### 5. Use Transactions Wisely

Migrations run in transactions by default. Be aware of database-specific transaction limitations:

```php
public function up(): void
{
    // These execute in a single transaction
    $this->table('users')->addColumn('field', 'string')->update();
    
    // Data operations also participate in the transaction
    $this->database()->insert('users')->values([...])->run();
    
    // If any operation fails, everything rolls back
}
```

### 6. Test Migrations in Development

Always test both `up()` and `down()` operations:

```php
// Run migration
$migrator->run();

// Verify changes
// ... test your application ...

// Test rollback
$migrator->rollback();

// Verify rollback worked correctly
// ... test your application ...

// Run again to ensure it's repeatable
$migrator->run();
```

### 7. Never Modify Published Migrations

Once a migration is committed and shared with your team or deployed to production, never modify it. Instead, create a
new migration:

```php
// Don't modify existing CreateUsersTableMigration

// Instead, create new migration
class AlterUsersTableAddPhoneNumberMigration extends Migration
{
    public function up(): void
    {
        $this->table('users')
            ->addColumn('phone', 'string', ['nullable' => true])
            ->update();
    }
}
```

### 8. Document Complex Operations

Add comments for non-obvious operations:

```php
public function up(): void
{
    // Convert role_id to role enum for better type safety
    // Old data: 1=admin, 2=editor, 3=user
    $this->table('users')
        ->addColumn('role_new', 'enum', [
            'values' => ['admin', 'editor', 'user'],
            'default' => 'user'
        ])
        ->update();
    
    // Migrate data from old column
    $this->database()->execute(
        "UPDATE users SET role_new = CASE role_id 
            WHEN 1 THEN 'admin'
            WHEN 2 THEN 'editor'
            ELSE 'user'
        END"
    );
    
    $this->table('users')
        ->dropColumn('role_id')
        ->renameColumn('role_new', 'role')
        ->update();
}
```

### 9. Consider Performance

For large tables, some operations can be slow:

```php
// Adding non-nullable columns to large tables
public function up(): void
{
    // Good - Add as nullable first, fill data, then make required
    $this->table('large_table')
        ->addColumn('new_field', 'string', ['nullable' => true])
        ->update();
    
    // Fill data in batches if needed
    
    // Then alter to non-nullable
    $this->table('large_table')
        ->alterColumn('new_field', 'string', ['nullable' => false])
        ->update();
}
```

### 10. Database-Specific Considerations

Be aware of database-specific limitations:

```php
// Some operations can't run in transactions on certain databases
// For example, ALTER TABLE on MySQL with certain engines

public function up(): void
{
    // This is safe - Cycle handles transaction management
    $this->table('users')
        ->addColumn('field', 'string')
        ->update();
}
```

## Compatibility with DBAL

All migration methods are built on top of DBAL functionality. You can use the same abstract column types as in direct
schema declarations.

Available abstract types:

- `primary`, `bigPrimary` - Auto-incrementing primary keys
- `boolean` - Boolean values
- `integer`, `bigInteger`, `tinyInteger` - Integer types
- `string`, `text`, `longText` - String types
- `float`, `double`, `decimal` - Floating-point types
- `datetime`, `date`, `time`, `timestamp` - Date/time types
- `binary`, `longBinary` - Binary data
- `json` - JSON data
- `enum` - Enumeration types
- `uuid` - UUID type (database-specific)

> Read more about column types and options in the [Schema Declaration](/docs/en/database/declaration.md) documentation.

## Integration with Cycle ORM

Cycle ORM can automatically generate migrations from your entity definitions. The schema compiler analyzes your entities
and creates appropriate migrations to synchronize the database.

**Workflow:**

1. Define entities using attributes or manual schema
2. Run schema compilation with migration generator
3. Review generated migrations
4. Execute migrations to update database
5. Repeat as schema evolves

This approach ensures your database schema stays synchronized with your entity definitions while maintaining full
control over migration execution.

> Read more about entity definitions in the [Annotated Entities](/docs/en/annotated/entity.md) documentation.
