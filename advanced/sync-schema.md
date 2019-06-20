# Syncronizing Database Schema
Cycle ORM provides multiple way to automatically configure database schema based on entity declarations.

## Automatic Syncronization
First approach would involve automatic schema declaration without any intermediate migration, to use it add `SyncTables` generetator to your schema compiler:

```php
$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Embeddings($cl),            // register embeddable entities
    new Annotated\Entities($cl),              // register annotated entities
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

Such approach is useful for development enviroment but might cause issues while working with production database.

## Generate Migrations
You can automatically generate set of migration files during schema compilaration. In this case you have a freedom to alter such migrations manually before running them. To achive that you must connect Cycle extension:

```php
composer require cycle/migrations
```

Migrations are based on `Spiral/Migrations` package and require proper configuration first:

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

You can now add new Compiler generator to render schema changes into migration files:

```php
$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Embeddings($cl),                                         // register embeddable entities
    new Annotated\Entities($cl),                                           // register annotated entities
    new Schema\Generator\ResetTables(),                                    // re-declared table schemas (remove columns)
    new Schema\Generator\GenerateRelations(),                              // generate entity relations
    new Schema\Generator\ValidateEntities(),                               // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),                                   // declare table schemas
    new Schema\Generator\RenderRelations(),                                // declare relation keys and indexes
    new \Cycle\Migrations\GenerateMigrations($migrator->getRepository()),  // sync table changes to database
    new Schema\Generator\GenerateTypecast(),                               // typecast non string columns
]);
```

> Make sure to remove `SyncTables`.

## Run Migrations
To get list of available migrations:

```php
print_r($migrator->getMigrations());
```

To run all outstanding migration:

```php
while($migrator->run() !== null) { }
```

To rollback last migration:

```php
$migrator->rollback();
```
