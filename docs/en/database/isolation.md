# Database - Logical Isolation

The `cycle/database` component provides the ability to use a single connection for multiple, logically isolated
databases. The isolation performed using table prefix, which automatically added to every SQL identifier.

## Configuration

To enable database prefix use option `prefix` of your database:

```php
<?php

declare(strict_types=1);

use Cycle\Database\DatabaseManager;
use Cycle\Database\Config;

$dbal = new DatabaseManager(new Config\DatabaseConfig([
    'databases'   => [
        'default' => ['connection' => 'sqlite', 'prefix' => 'my_prefix_'],
    ],
    'connections' => [
        'sqlite' => new Config\SQLiteDriverConfig(
            connection: new Config\SQLite\MemoryConnectionConfig(),
            queryCache: true,
        ),
    ],
]));
```

## Runtime configuration

You can isolate database in runtime using `withPrefix` method:

```php
$db = $dbal->database();
print_r($db->withPrefix('my_db_prefix')->getTables());
```

> The schema introspection and declaration will work with the isolated database by automatically adding a prefix to 
> all declared tables.
