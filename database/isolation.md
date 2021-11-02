# Database - Logical Isolation

The `cycle/database` component provides the ability to use a single connection for multiple, logically isolated databases.
The isolation performed using table prefix, which automatically added to every SQL identifier.

## Configuration
To enable database prefix use option `prefix` of your database:

```php
<?php

declare(strict_types=1);

use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\DatabaseManager;
use Cycle\Database\Driver\SQLite\SQLiteDriver;

$dbal = new DatabaseManager(new DatabaseConfig([
    'databases'   => [
        'default' => ['connection' => 'sqlite', 'prefix' => 'my_prefix_'],
    ],
    'connections' => [
        'sqlite' => [
            'driver'     => SQLiteDriver::class,
            'connection' => 'sqlite:database.db',
        ],
    ],
]));
```

## Runtime configuration
You can isolate database in runtime using `withPrefix` method:

```php
$db = $dbal->database();
var_dump($db->withPrefix('my_db_prefix')->getTables());
```

> The schema introspection and declaration will work with the isolated database by automatically adding a prefix to all declared
> tables.
