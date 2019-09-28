# Installation
Cycle ORM can be installed into any PHP application using `composer` dependency manager.

## Requirements
  * PHP 7.2+
  * PHP-PDO
  * PDO drivers for desired databases
  
   
## Installation
Cycle ORM is available as composer repository and can be installed using the following command in the root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request an additional package:

```bash
$ composer require cycle/annotated
```

This command will also download Cycle ORM dependencies such as `spiral/database`, `doctrine/collections` and `zendframework/zend-hydrator`.

In order to access Cycle ORM classes make sure to include `vendor/autoload.php` in your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Connect Database
In order to connect Cycle to the proper database instance, you must configure the instance of `Spiral\Database\DatabaseManager`. 
The details of configuration process described in a [following section](/basic/connect.md).

## Instantiate ORM
An ORM service can be instantiated using `Cycle\ORM\ORM` class which takes only one dependency on `Cycle\ORM\Factory`:

```php
$orm = new ORM\ORM(new ORM\Factory($dbal));
```

## Schema Generation (Schema Update)
In order to operate Cycle ORM require schema definition. The schema will contain a list of available entities, their relations, and association with a specific database. 

The schema can be descibed manually by instatiating `Cycle\ORM\Schema` object:

```php
$schema = new Schema([
    'user' => [
        Schema::ENTITY      => User::class,
        Schema::MAPPER      => Mapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => [
            'id'      => 'id', 
            'email'   => 'email',
            'balance' => 'balance'
        ],
        Schema::TYPECAST    => [
            'id'      => 'int',
            'balance' => 'float'
        ],
        Schema::SCHEMA      => [],
        Schema::RELATIONS   => [
            'address' => [
                 Relation::TYPE   => Relation::HAS_ONE,
                 Relation::TARGET => 'address',
                 Relation::SCHEMA => [
                     Relation::CASCADE   => true,
                     Relation::INNER_KEY => 'id',
                     Relation::OUTER_KEY => 'user_id',
                 ],
            ]
        ]
    ],
    'address' => [
        Schema::ENTITY      => Address::class,
        Schema::MAPPER      => Mapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'address',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => [
            'id'      => 'id', 
            'user_id' => 'user_id',
            'city'    => 'city'
        ],
        Schema::TYPECAST    => [
            'id'      => 'int'
        ],
        Schema::SCHEMA      => [],
        Schema::RELATIONS   => []
    ],
]);

$orm = $orm->withSchema($schema);
```

However, in order to simplify the integration, it is recommended to use a schema compiler provided by `cycle/schema-builder` extension. Such compiler is able to automatically index all available entities, perform database introspection and reflection. 

To compile the schema using annotated entities and automatically configure the database using the following pipeline:

```php
use Cycle\Schema;
use Cycle\Annotated;
use Spiral\Tokenizer;

// Class locator
$cl = (new Tokenizer\Tokenizer(new Tokenizer\Config\TokenizerConfig([
    'directories' => ['src/'],
])))->classLocator();

$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Embeddings($cl),            // register embeddable entities
    new Annotated\Entities($cl),              // register annotated entities
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Annotated\MergeColumns(),             // copy column declarations from all related classes (@Table annotation)
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Annotated\MergeIndexes(),             // copy index declarations from all related classes (@Table annotation)
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);

$orm = $orm->withSchema(new Schema($schema));
```

The result of schema builder has compiled schema, given schema can be cached in order to avoid expensive calculations on each request.

> In the following section the terming `update schema` will be referenced to this process.

Remove element `new Schema\Generator\SyncTables()` to disable database reflection. In a later section, we will describe how to automatically render database migrations instead of direct synchronization. 
