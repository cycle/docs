# Connect to Database
Cycle ORM requires at least one connection to the database in order to operate. The DBAL functionality is
provided by the package `spiral/database`.

> Make sure to install all required dependencies listed in the previous section.

## Instantiate DBAL
In order to start, we have to initialize the `DatabaseManager` service used to automatically create and manage a set of application databases.
The list of available connections and databases can be listed in the initial configuration.

```php
$dbConfig = new Database\Config\DatabaseConfig([
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
]);

$dbal = new Database\DatabaseManager($dbConfig);
```

> You can instantiate DBAL with an empty connection list and configure it in runtime if needed.

## Configure Databases
The Spiral/Database module provides support to manage multiple databases in one application, use read/write connections and logically
separate multiple databases within one connection using prefixes.

To register a new database simply add it into `databases` section:

```php
'default' => [
  'connection' => 'sqlite'
  'driver': 'sqlite'
]
```

To use a database-specific prefix use the `prefix` option (all the queries will be affected):

```php
'default' => [
  'connection' => 'sqlite',
  'prefix'     => 'secondary_'
]
```

To use read/write connections use sections `connection` and `readConnection` accordingly:

```php
'default' => [
  'connection'     => 'mysql',
  'readConnection' => 'mysqlSlave',
  'prefix'         => 'secondary_'
]
```

## Connections
Each database instance must have an associated connection object. Connections used to provide low-level functionality and wrap
different database drivers. To register new a connection you have to specify the driver class and its connection options:

For **SQLite**:

```php
'sqlite' => [
    'driver'  => Driver\SQLite\SQLiteDriver::class,
    'options' => [
        'connection' => 'sqlite:database.db',
        'username'   => '',
        'password'   => '',
    ]
]
```

> Use `sqlite::memory:` to keep SQLite in memory.

For `MySQL` and `MariaDB`:

```php
'mysql'     => [
  'driver'  => Driver\MySQL\MySQLDriver::class,
  'options' => [
    'connection' => 'mysql:host=127.0.0.1;dbname=database',
    'username'   => 'mysql',
    'password'   => 'mysql',
  ]
],
```

For `PostgresSQL`:

```php
'postgres'  => [
  'driver'   => Driver\Postgres\PostgresDriver::class,
  'options' => [
      'connection' => 'pgsql:host=127.0.0.1;dbname=database',
      'username'   => 'postgres',
      'password'   => 'postgres',
   ],
],
```

For `SQLServer`:

```php
'sqlServer' => [
  'driver'  => Driver\SQLServer\SQLServerDriver::class,
  'options' => [
    'connection' => 'sqlsrv:Server=OWNER;Database=DATABASE',
    'username'   => 'sqlServer',
    'password'   => 'sqlServer',
  ],
],
```
> Make sure to install the proper PDO extensions!

## Additional connection options
There are multiple connection options you can use to customize the behavior.

Options | Value | Description
--- | --- | ---
timezone | string | Default driver timezone (all DateTimeInterface query parameters will be converted into it). Defaults to `UTC`.
reconnect | bool | Allow the driver to automatically reconnect. Defaults to `false`.
profiling | bool | Enable SQL profiling (logging). Defaults to `false`.

## Access Database
To access the database using the `DatabaseManager`, use the method `database`:

```php
print_r($dbal->database('default'));
```

The database will be automatically connected on the first SQL request.

> DBAL will use the database specified in the `default` config option if the name is `null`.

Direct SQL queries as possible from this moment:

```php
$dbal->database('default')->table('users')->select()->fetchAll();
```

## Profiling and Logging
Each of the database drivers implements the `Psr\Log\LoggerAwareInterface`. You can enable SQL logging by assigning a logger :

```php
$driver = $dbal->database('default')->getDriver();
$driver->setLogger($myLogger);
```

## Runtime Configuration
In addition to config driven setup you are able to configure your database connections in runtime:

```php
$dbal->addDatabase(new Database(
  'name',
  'prefix_',
  new SQLiteDriver(
     'connection'  => 'sqlite::memory:',
     'username'   => 'username',
     'password'   => 'password',
     'options'    => []
  )
));
```

> This approach can be useful to test your application using database mocks. Attention, DBAL would not allow you to overwrite already exists database, you must explicitly configure empty `DatabaseManager`.
