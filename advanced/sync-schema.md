# Synchronizing Database Schema
Cycle ORM provides multiple ways to automatically configure your database schema based on entity declarations.

## Automatic Synchronization
A first approach would involve automatic schema declaration without any intermediate migration, to use it add `SyncTables` generator to your schema compiler:

```php
$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Annotated\Embeddings($cl),            // register embeddable entities
    new Annotated\Entities($cl),              // register annotated entities
    new Annotated\MergeColumns(),             // add @Table column declarations
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Annotated\MergeIndexes(),             // add @Table column declarations
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

Such an approach is useful for development environments, but might cause issues while working with the production database.

## Generate Migrations
You can automatically generate a set of migration files during schema compilation. In this case, you have the freedom to alter such migrations manually before running them. To achieve that you must install the Cycle Migrations extension:

```php
composer require cycle/migrations
```

Migrations are based on the `Spiral/Migrations` package and require proper configuration first:

```php
use Spiral\Migrations;

$config = new Migrations\Config\MigrationConfig([
    'directory' => __DIR__ . '/../migrations/',  // where to store migrations
    'table'     => 'migrations'                 // database table to store migration status
]);

$migrator = new Migrations\Migrator($config, $dbal, new Migrations\FileRepository($config));

// Init migration table
$migrator->configure();
```

You can now add a new Compiler generator to render schema changes into migration files:

```php
$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Schema\Generator\ResetTables(),                                    // re-declared table schemas (remove columns)
    new Annotated\Embeddings($cl),                                         // register embeddable entities
    new Annotated\Entities($cl),                                           // register annotated entities
    new Annotated\MergeColumns(),                                          // add @Table column declarations
    new Schema\Generator\GenerateRelations(),                              // generate entity relations
    new Schema\Generator\ValidateEntities(),                               // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),                                   // declare table schemas
    new Schema\Generator\RenderRelations(),                                // declare relation keys and indexes
    new Annotated\MergeIndexes(),                                          // add @Table column declarations
    new \Cycle\Migrations\GenerateMigrations($migrator->getRepository()),  // generate migrations
    new Schema\Generator\GenerateTypecast(),                               // typecast non-string columns
]);
```

> Make sure to remove `SyncTables`.

## Run Migrations
To get a list of available migrations:

```php
print_r($migrator->getMigrations());
```

To run all outstanding migrations:

```php
while($migrator->run() !== null) { }
```

To rollback the last migration:

```php
$migrator->rollback();
```
