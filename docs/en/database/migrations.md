# Database - Migrations

Migrations are a convenient way for you to alter your database in a structured and organized manner. This package adds
additional functionality for versioning your database schema and easily deploying changes to it. It is a very easy to
use and a powerful tool.

## Installation

```bash
composer require cycle/migrations
```

## Configure Migrations (optional)

You can configure what database and table to use to store information about the schema version in migrations config.

```php
use Cycle\Migrations;

$config = new Migrations\Config\MigrationConfig([
    'directory' => __DIR__ . '/../migrations/', // where to store migrations
    'table'     => 'migrations',                // database table to store migration status
    'safe'      => true                         // When set to true no confirmation will be requested on migration run.
]);

$migrator = new Migrations\Migrator($config, $dbal, new Migrations\FileRepository($config));

// Init migration table
$migrator->configure();
```

## Generate Migrations

You can automatically generate a set of migration files during schema compilation. In this case, you have the freedom to
alter such migrations manually before running them. To achieve that you must install the Cycle Migrations extension:

```bash
composer require cycle/schema-migrations-generator
```

```php
use Cycle\Schema\Registry;
use Cycle\Schema\Generator\Migrations;
use Cycle\Schema\Definition\Entity;
use Cycle\Schema\Generator\Migrations\GenerateMigrations;

$registry = new Registry($dbal);
$registry->register(....);

$generator = new GenerateMigrations($migrator->getRepository(), $migrator->getConfig());

// Migration generator creates set of migrations needed to sync database schema with desired state.
// Each database will receive it's own migration.
$generator->run($registry);
```

### Migration strategies

The migration generator can use different strategies to generate migrations. The default strategy is to generate a
migration for each database. You can change this behavior by passing a different strategy to the generator. The package
provides the following strategies:

- `Cycle\Schema\Generator\Migrations\Strategy\SingleFileStrategy` - generates a migration file for each database.
   This is the default strategy.
- `Cycle\Schema\Generator\Migrations\Strategy\MultipleFilesStrategy` - generates separate migration files for each table,
   offering improved organization and readability.

The preferred strategy can be passed as a parameter when creating the `GenerateMigrations` object. For example,
let's change the default `SingleFileStrategy` to `MultipleFilesStrategy`:

```php
use Cycle\Schema\Generator\Migrations\GenerateMigrations;
use Cycle\Schema\Generator\Migrations\Strategy\MultipleFilesStrategy;
use Cycle\Schema\Generator\Migrations\NameBasedOnChangesGenerator;

$generator = new GenerateMigrations(
    $migrator->getRepository(),
    $migrator->getConfig(),
    new MultipleFilesStrategy($migrator->getConfig(), new NameBasedOnChangesGenerator())
);
```

The second parameter in the migration strategy is `Cycle\Schema\Generator\Migrations\NameGeneratorInterface`,
which is the migration name generator. The package provides one implementation of this interface -
`Cycle\Schema\Generator\Migrations\NameBasedOnChangesGenerator`.

## Create a migration

We can create our migrations manually:

```php
class MyMigrationMigration extends Migration
{
    /**
     * Create tables, add columns or insert data here
     */
    public function up()
    {
        $this->table('sample_table')
            ->addColumn('id', 'primary')
            ->addColumn('name', 'string')
            ->create();
    }

    /**
     * Drop created, columns and etc here
     */
    public function down()
    {
        $this->table('sample_table')->drop();
    }
}
```

## Working with migrations

You can run all outstanding migrations using `migrate` command.

```php
use Cycle\Migrations\Capsule;

$migrator->run(new Capsule($dbal->database()));
```

```sql
Rolling back executed migration(s)...
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
[MySQLDriver] Begin transaction
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'sample_table'
[MySQLDriver] SHOW FULL COLUMNS FROM `sample_table`
[MySQLDriver] SHOW INDEXES FROM `sample_table`
[MySQLDriver] SELECT * FROM `information_schema`.`referential_constraints` WHERE `constraint_schema` = 'sample_2' AND `table_name` = 'sample_table'
[MySQLDriver] SHOW INDEXES FROM `sample_table`
[MySQLDriver] SHOW TABLE STATUS WHERE `Name` = 'sample_table'
[MySQLDriver] DROP TABLE `sample_table`
[MySQLDriver] Commit transaction
[MySQLDriver] DELETE FROM `migrations`
WHERE `migration` = 'my_migration'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
Migration my_migration was successfully rolled back.

Executing outstanding migration(s)...
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
[MySQLDriver] Begin transaction
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'sample_table'
[MySQLDriver] CREATE TABLE `sample_table` (
    `id` int (11) NOT NULL AUTO_INCREMENT,
    `name` varchar (255) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE InnoDB
[MySQLDriver] Commit transaction
[MySQLDriver] INSERT INTO `migrations` (`migration`, `time_executed`)
VALUES ('my_migration', '2017-04-01 16:12:21')
[MySQLDriver] Given insert ID: 2
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
Migration my_migration was successfully executed.
```

## Update existed schema

Create migrations to alter existed table schema:

```php
class NewFieldMigration extends Migration
{
    public function up()
    {
        $this->table('sample_table')
            ->addColumn('field', 'float')
            ->update();
    }

    public function down()
    {
        $this->table('sample_table')
            ->dropColumn('field')
            ->update();
    }
}
```

```sql
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'new_field'
[MySQLDriver] Begin transaction
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'sample_table'
[MySQLDriver] SHOW FULL COLUMNS FROM `sample_table`
[MySQLDriver] SHOW INDEXES FROM `sample_table`
[MySQLDriver] SELECT * FROM `information_schema`.`referential_constraints` WHERE `constraint_schema` = 'sample_2' AND `table_name` = 'sample_table'
[MySQLDriver] SHOW INDEXES FROM `sample_table`
[MySQLDriver] SHOW TABLE STATUS WHERE `Name` = 'sample_table'
[MySQLDriver] ALTER TABLE `sample_table` ADD COLUMN `field` float NOT NULL
[MySQLDriver] Commit transaction
[MySQLDriver] INSERT INTO `migrations` (`migration`, `time_executed`)
VALUES ('new_field', '2017-04-01 16:23:36')
[MySQLDriver] Given insert ID: 4
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'new_field'
Migration new_field was successfully executed.
[MySQLDriver] SELECT COUNT(*) FROM `information_schema`.`tables` WHERE `table_schema` = 'sample_2' AND `table_name` = 'migrations'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'my_migration'
[MySQLDriver] SELECT
`id`, `time_executed`
FROM `migrations`
WHERE `migration` = 'new_field'
```

## Compatibility with DBAL

All migration methods are based on DBAL functions, feel free to use same abstract types as in
[direct schema declarations](/docs/en/database/declaration.md).

> **Note**
> Note that the Cycle ORM component can create migrations automatically.
