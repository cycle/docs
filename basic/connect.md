# Connect to Database
Cycle ORM required at least one connection to database in order to operate. The DBAL functionaltiy is
provided by the package `spiral/database`.

> Make sure to install all required dependencies listed in previous section.

## Instantiate DBAL
In order to start we have to initialize `DatabaseManager` service used to automatically create and manage set of application databases.
The list of available connections and databases can be listed in initial configuration.

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

> You can instantiate DBAL with empty connection list and configure it in runtime if needed.

## Configure Databases
Spiral/Database module provide support to manage multiple databases in one application, use read/write connections and logically 
separate multiple databases within one connection using prefixes.

To register new database simply add it into `databases` section:

```php
'default' => [
  'connection' => 'sqlite'
]
```

To give database specific prefix use `prefix` option (all the queries will be affected):

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
Each of database instance must have associated connection object. Connections used to provide low level functionality and wrap
different database drivers. To register new connection you have to specify connection class and it's connection options:

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

For `PosgresSQL`:

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
> Make sure to instance proper PDO extension!

## Additional connection options
There is multiple connection options you can use to customize the behaviour.

Options | Value | Description 
--- | --- | ---
timezone | string | Default driver timezone (all DateTimeInterface objects will be converted into it), defaults to `UTC`.
reconnect | bool | Allow driver to automatically reconnect, defaults to `false`.
profiling | bool | Enable SQL profiling (logging), defaults to `false`.

## Access Database
To access database using `DatabaseManager` use method `database`:

```php
print_r($dbal->database('default'));
```

Database will be automatically connected on first SQL request.

> DBAL will use database specified in `default` config option if name is `null`.

Direct SQL queries as possible from this moment:

```php
$dbal->database('default')->table('users')->select()->fetchAll();
```

## Profiling and Logging
Each of database Driver implements `Psr\Log\LoggerAwareInterface`, you can enable SQL logging by assigning logger and enabling Driver 
profiling:

```php
$driver = $dbal->database('default')->getDriver();
$driver->setLogger($myLogger);
$driver->setProfiling(true);
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

> This approach can be useful to test your application using database mocks. Attention, DBAL would not allow you to overwrite already 
exists, you must explicitly configure empty `DatabaseManager`.
database 
