# Database - Errata

The documentation section lists non-obvious behaviors of the DBAL component and ways to avoid them.

## Timestamps

Consider using 'datetime' type for your columns instead of 'timestamp' as it gives you more unified support and ability
to define proper default values.

> Timestamp fields behavior is different in MySQL 5.5 and MySQL 5.6+!

## Driver Specific notes

Not all drivers implemented equally:

### MySQL Driver

The DBAL layer fully supports MySQL driver with all functionality being available.

The default table engine set to `InnoDB`.

### Postgres Driver

Postgres driver includes custom implementation of `InsertQuery` to address the return value of auto-incremental PK, it
will automatically add `RETURNING {primary key}` to the generated SQL query.

#### Postgres schemas

By default, Postgres driver uses `public` schema and works only with tables with `public` schema. If you want to change 
default behavior, you have to pass desired schema list via connection config:

```php
'postgres' => new Config\PostgresDriverConfig(
    connection: new Config\Postgres\TcpConnectionConfig(
        database: 'spiral',
        host: '127.0.0.1',
        port: 5432,
        user: 'spiral',
        password: '',
    ),
    schema: [
        'public', 
        'private', 
        '$user' // Current user namespace
    ],
    queryCache: true,
),
```

Given list of schemas will be used for filtering available tables. 

##### Schema declaration
```php
// will be used the first schema from config (public)
$schema1 = $dbal->database()->table('users')->getSchema();
$schema1->column('id')->primary();
$schema1->save();

// private
$schema = $dbal->database()->table('private.users')->getSchema();
$schema->column('id')->primary();
$schema->save();

// test
$schema2 = $dbal->database()->table('test.users')->getSchema();
$schema2->column('id')->primary();
$schema2->save();

// Current user schema
$schema2 = $dbal->database()->table('$user.users')->getSchema();
$schema2->column('id')->primary();
$schema2->save();
// ...
```

##### Database hasTable method

```php
$db = $dbal->database();

$db->hasTable('private.users'); // true

$db->hasTable('test.users'); // true

// Current user schema
$db->hasTable('$user.users'); // true

// will be used the first schema from config (public)
$db->hasTable('users'); // true
// ...
```

##### Database getTableNames method

Method will return tables with schema.

```php
$db = $dbal->database();
$tables = $db->getTables();

$tables[0]->getName(); // users
$tables[0]->getFullName(); // private.users

$tables[1]->getName(); // users
$tables[1]->getFullName(); // private.users

$tables[2]->getName(); // users
$tables[2]->getFullName(); // spiral.users - current user namespace for user with username 'spiral'

// table with schema 'test' will be ignored. It doesn't present in connection config schema list

// ...
```

##### Query builder

You can use table names with schema declaration in a Query builder via dot notation.

```php
$db = $dbal->database();
$tables = $db->select()->from('private.users')->...
```

### SQLServer Driver

SQLServer includes a fallback mechanism to limit your selection without orderBy specified. In some cases, it might add
`_row_number_` column at the end of the selected columns.

> SQLServer is optimized to work with 12+ versions of [Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server/).

### SQLite Driver

SQLite DBMS does not support a set of table altering operations, to address some of the schema migrations will be
performed using temporary tables and data copy.
