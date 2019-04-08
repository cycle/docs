# Installation
Cycle ORM requiments include:
  * PHP7.1+
  * PHP-PDO
  * PDO drivers for desired databases

## Installation
Cycle ORM is available as composer repository and can be installed using following command in a root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request additional package:

```bash
$ composer require cycle/annotated
```

Given commands will download Cycle ORM dependencies such as `spiral/databses`, `doctrine/collections` and `zendframework/zend-hydrator`.

In order to access to Cycle ORM classes make sure to include `vendor/autoload.php` into your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Configuration
In order to operate, Cycle ORM require proper database connection to be set. All database connections are managed using `DatabaseManager` service provided by the package `spiral/database`. We can configure our first database connection to be initiated on demand using following configuration:

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';

$dbal = new Database\DatabaseManager(new Database\Config\DatabaseConfig([
    'default'     => 'default',
    'databases'   => [
        'default' => [
            'connection' => 'sqlite'
        ]
    ],
    'connections' => [
        'sqlite' => [
            'driver'  => Database\Driver\SQLite\SQLiteDriver::class,
            'options' => [
                'connection' => 'sqlite:database.db',
                'username'   => '',
                'password'   => '',
            ]
        ]
    ]
]));
```

> Read about how to connect to other database types in [next section](connection.md). You can also configure
database connections in runtime.

We can get access to our database object and check connection:

```php
print_r($dbal->database('default')->getTables());
```

> Run `php filename.php`, the result must be empty array.

## ORM
ORM service can be initiated using following construction:

```php
$orm = new ORM\ORM(new ORM\Factory(
    $dbal,
    ORM\Config\RelationConfig::getDefault()
));
```

> Make sure to add `use Cycle\ORM;` at top of your file.


