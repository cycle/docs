# Database - Installation and Configuration
The `cycle/database` component is included by default in Web and GRPC builds. The DBAL focuses mainly on unifying database access rather than trying to get 100% of the specific DBMS feature set.
 
However, you can always use direct queries to bypass the spiral abstractions.

> Spiral DBAL supports MySQL, MariaDB, SQLite, PostgresSQL, and SQLServer (Windows) databases.

## Installation
To install the component in alternative bundles or as a standalone library: 

```bash
$ composer require cycle/database
```

### Declare Connection
To create new database connection add a new section or alter existed options of `drivers` section of your configuration, 
you can use `env` function to keep your passwords and usernames separately.

```php
<?php

require_once "vendor/autoload.php";

use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\DatabaseManager;
use Cycle\Database\Driver;

$dbal = new DatabaseManager(new DatabaseConfig([
    'databases' => [
        'default' => ['driver' => 'runtime'],
    ],
    'connections' => [
        'mysql'     => [
            'driver'     => Driver\MySQL\MySQLDriver::class,
            'options'    => [
                'connection' => 'mysql:host=127.0.0.1;dbname=' . env('DB_NAME'),
                'username'   => 'username',
                'password'   => 'password',
            ]
        ],
        'postgres'  => [
            'driver'     => Driver\Postgres\PostgresDriver::class,
            'options'    => [
                'connection' => 'pgsql:host=127.0.0.1;dbname=' . env('DB_NAME'),
                'username'   => 'username',
                'password'   => 'password',
            ]
        ],
        'runtime'   => [
            'driver'     => Driver\SQLite\SQLiteDriver::class,
            'options'    => [
                'connection' => 'sqlite:' . directory('runtime') . 'runtime.db',
                'username'   => 'sqlite',
                'password'   => '',
            ]
        ],
        'sqlServer' => [
            'driver'     => Driver\SQLServer\SQLServerDriver::class,
            'options'    => [
                'connection' => 'sqlsrv:Server=MY-PC;Database=' . env('DB_NAME'),
                'username'   => 'username',
                'password'   => 'password',
            ]
        ]
    ],
]));
```

> Use connection option `options` to set PDO specific attributes.

### Declare Database
In order to access connected database we have to add it into `databases` section first:

```php
<?php

declare(strict_types=1);

require_once "vendor/autoload.php";

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

var_dump($dbal->database('primary'));

var_dump($dbal->database('secondary'));
```

### Aliases
Your application and modules can access the database in multiple different ways. Database aliasing allows you to use
separate databases with relation to one physical database.

```php
<?php

declare(strict_types=1);

require_once "vendor/autoload.php";

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

var_dump($dbal->database('db'));

var_dump($dbal->database('other'));
```

To create Database instance manually (without the DatabaseManager):

```php

use Cycle\Database\Driver;

$driver = new Driver\SQLite\SQLiteDriver(
   [
       'connection' => 'sqlite:db.db'
   ]
);
       
$db = new Database\Database(
   'name',
   '',
   $driver,
   $driver // read only driver (optional)
);

var_dump($db->getTables());
```
