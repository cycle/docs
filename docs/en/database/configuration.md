# Database - Installation and Configuration

The `cycle/database` component is included by default in Web and GRPC builds. The DBAL focuses mainly on unifying
database access rather than trying to get 100% of the specific DBMS feature set.

However, you can always use direct queries to bypass the spiral abstractions.

> Spiral DBAL supports MySQL, MariaDB, SQLite, PostgresSQL, and SQLServer (Windows) databases.

## Installation

To install the component in alternative bundles or as a standalone library:

```bash
composer require cycle/database
```

### Declare Connection

To create new database connection add a new section or alter existed options of `drivers` section of your configuration,
you can use `env` function to keep your passwords and usernames separately.

```php
<?php

require_once "vendor/autoload.php";

use Cycle\Database\DatabaseManager;
use Cycle\Database\Driver;
use Cycle\Database\Config;

$dbal = new DatabaseManager(new Config\DatabaseConfig([
    'databases' => [
        'default' => [
            'driver' => 'runtime'
        ],
    ],
    'connections' => [
        'runtime' => new Config\SQLiteDriverConfig(
            connection: new Config\SQLite\FileConnectionConfig(
                database:  __DIR__.'./runtime/database.sqlite'
            ),
            queryCache: true,
        ),
    ],
]));
```

> Read about how to connect to other database types in [this section](/docs/en/database/connect.md)

### Declare Database

In order to access connected database we have to add it into `databases` section first:

```php
use Cycle\Database\Driver;

$dbal = new DatabaseManager(new DatabaseConfig([
    'databases'   => [
        'primary' => [
           'driver'  => 'mysql',
           'prefix'  => 'primary_'
        ],
        'secondary'=> [
            'driver'  => 'mysql',
            'prefix'  => 'secondary_'
        ]
    ],
    'connections' => [
        // ...
    ],
]));

print_r($dbal->database('primary'));

print_r($dbal->database('secondary'));
```

### Aliases

Your application and modules can access the database in multiple different ways. Database aliasing allows you to use
separate databases with relation to one physical database.

```php
use Cycle\Database\Driver;

$dbal = new DatabaseManager(new DatabaseConfig([
    'aliases'   => [
        'db'    => 'primary',
        'other' => 'secondary'
    ],
    'databases'   => [
        // ...
    ],
    'connections' => [
        // ...
    ],
]));

print_r($dbal->database('db'));

print_r($dbal->database('other'));
```

### Read / Write Connections

Sometimes you may wish to use one database connection for SELECT statements, and another for INSERT, UPDATE, and DELETE
statements.

To see how read / write connections should be configured, let's look at this example:

```php
use Cycle\Database\Driver;

$dbal = new DatabaseManager(new DatabaseConfig([
    'databases' => [
        // Will be used for read and write operations
        'primary' => [
           'driver' => 'mysql', 
           // or
           'write' => 'mysql'
           
           // ...
        ],
        'secondary' => [
            'write' => 'mysql', // Will be used for write operations
            'read' => 'sqlite', // Will be used for read operations
             // ...
        ]
    ],
    'connections' => [
        // ...
    ],
]));
```

### Manual Database instance creation

To create Database instance manually (without the DatabaseManager):

```php

use Cycle\Database\Driver;
use Cycle\Database\Config;

$writeDriver = Driver\SQLite\SQLiteDriver::create(
    new Config\MemoryConnectionConfig()
);

$readDriver = Driver\SQLite\SQLiteDriver::create(
    new Config\TempFileConnectionConfig()
);
       
$db = new Database\Database(
   name: 'name',
   prefix: '',
   driver: $writeDriver,
   readDriver: $readDriver // read only driver (optional)
);

print_r($db->getTables());
```
