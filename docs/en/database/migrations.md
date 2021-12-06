# Database - Migrations
Spiral ships with a set of embedded commands to control your database migrations, [component](https://github.com/cycle/migrations) 
is build upon DBAL and supports virtual databases and prefixes.

```php
composer require cycle/migrations
```

## Configure Migrations (optional)
You can configure what database and table to use to store information about the schema version in migrations config.

```php
use Cycle\Migrations;

$config = new Migrations\Config\MigrationConfig([
    'directory' => __DIR__ . '/../migrations/',    // where to store migrations
    'table'     => 'migrations'                    // database table to store migration status
    'safe'      => true                            // When set to true no confirmation will be requested on migration run. 
]);

$migrator = new Migrations\Migrator($config, $dbal, new Migrations\FileRepository($config));

// Init migration table
$migrator->configure();
```

## Generate Migrations
You can automatically generate a set of migration files during schema compilation. In this case, you have the freedom 
to alter such migrations manually before running them. To achieve that you must install the Cycle Migrations extension:

```php
composer require cycle/schema-migrations-generator
```

```php
use Cycle\Schema\Registry;
use Cycle\Schema\Generator\Migrations;
use Cycle\Schema\Definition\Entity;

$registry = new Registry($dbal);
$registry->register(....);

$generator = new Migrations\GenerateMigrations($migrator->getRepository(), $migrator->getConfig());

// Migration generator creates set of migrations needed to sync database schema with desired state.
// Each database will receive it's own migration.
$generator->run($registry);
```

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

```bash
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

```
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

> Note that the Cycle ORM component can create migrations automatically.
