# Installation
Cycle ORM can be installed into any PHP application using `composer` dependency manager.

## Requirements
  * PHP 8.0+
  * PHP-PDO
  * PDO drivers for desired databases

## Installation
Cycle ORM is available as a composer package and can be installed using the following command in the root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to install an additional package:

```bash
$ composer require cycle/annotated
```

This command will also download Cycle ORM dependencies such as `cycle/database`.

In order to access Cycle ORM classes make sure to include `vendor/autoload.php` in your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Connect Database
In order to connect Cycle to the proper database instance, you must configure the instance of `Cycle\Database\DatabaseManager`.
The details of this configuration process described in a [following section](/docs/en/database/connect.md).

## Instantiate ORM
An ORM service can be instantiated using the `Cycle\ORM\ORM` class, which takes only one dependency on `Cycle\ORM\Factory`:

```php
use Cycle\ORM;

$orm = new ORM\ORM(new ORM\Factory($dbal));
```

## Schema Generation (Schema Update)
In order to operate Cycle ORM a schema definition is required. The schema will contain a list of available entities, their relations, and association with a specific database.

The schema can be described manually by instantiating `Cycle\ORM\Schema` object:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Relation;
use Cycle\ORM\Mapper\Mapper;

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

$orm = $orm->with(schema: $schema);
```

However, in order to simplify the integration, it is recommended to use a schema compiler provided by `cycle/schema-builder` extension. 
Such compiler is able to automatically index all available entities, perform database introspection and reflection.

To compile the schema using annotated entities and automatically configure the database use the following [instruction](/docs/en/annotated/prerequisites.md).
