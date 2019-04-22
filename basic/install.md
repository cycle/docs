# Installation
Cycle ORM can be installed into any PHP application using `composer` dependency mananger.

## Requirements
  * PHP7.1+
  * PHP-PDO
  * PDO drivers for desired databases
  
   
## Installation
Cycle ORM is available as composer repository and can be installed using the following command in a root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request additional package:

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
In order to connect Cycle to proper database instance you must configure instance of `Spiral\Database\DatabaseManager`. 
The details of configuration process described in a [following section](basic/connect.md).

## Instantiate ORM
An ORM service can be instantiated using `Cycle\ORM\ORM` class which takes only one dependency on `Cycle\ORM\Factory`:

```php
$orm = new ORM\ORM(new ORM\Factory($dbal));
```

## Schema Generation (Schema Update)
In order to operate Cycle ORM require schema definition. The schema will contain list of available entities, their relations
and association with specific database. 

The schema can be descibed manually by instatiating `Cycle\ORM\Schema` object:

```php
$schema = new Schema([

]);

$orm = $orm->withSchema($schema);
```

However, in order to simplify the integration it is recommended to use schema compiler provided by `cycle/schema-builder` extenstion. Such compiler is able to automatically index all available entities, perform database introspection and reflection. 

To compile schema using annotated entities and automatically configure the database use followoing pipeline:

```php
use Cycle\Schema;
use Cycle\Annotated;

$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Entities($cl),              // register annotated entities
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Annotated\Columns(),                  // register non field columns (table level)
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Annotated\Indexes(),                  // register non entity indexes (table leve)
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

The result of schema builder is compiled schema, given schema can be cached in order to avoid expensive calculations on each request.

> In a following section the terming `update schema` will be referrenced to this process.

Remove element `new Schema\Generator\SyncTables()` to disable database reflection. In a later sections we will describe how to automatically render database migrations instead of direct synchronization. 
