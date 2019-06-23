# Dynamic Entity Schema
Cycle ORM does not limit developer ability to carry entity information using unique classes, it is possible to use specifically crafted model/models to represent entity data in it's own way. ORM support an ability to carry information using StdClass objects out of the box, it is also possible to combine classic annotated entities and StdClasses within one ORM scope.

You can also change ORM schema at runtime.

## StdMapper
In order to represent entity as StdClass you have to change it's mapper from default `Cycle\ORM\Mapper\Mapper` to `Cycle\ORM\Mapper\StdMapper`.
This can be done by either defining your schema using schema builder or by writing schema manually.

## Using Schema Builder
We can build our entities using entity definitions:

```php
//
// Lets create a new entity object.
//
$e = new Entity();
$e->setRole('user');
$e->setMapper(Cycle\ORM\Mapper\StdMapper::class);


//
// And now we will try to add to it a new field that 
// was not described in the original mapper.
//
$field = (new Field())
    ->setType('primary')
    ->setColumn('id')
    ->setPrimary(true);
    
$entity->getFields()->set('id', $field);


//
// Add entity definition into registry.
//
$registry = new Registry($this->dbal);
$registry->register($e)->linkTable($e, 'default', 'user');


//
// Compile the ORM schema.
//
$schema = (new Compiler())->compile($r, []);

print_r($schema);

$orm = $orm->withSchema(new Schema($schema));
```

> Use SyncTable [generator](/advanced/schema-builder.md) to update your database schema.

## Manual Schema Definition
You can also define StdClass schema manually using set of constants exposed by `Cycle\ORM\Schema` and `Cycle\ORM\Relation` classes:

```php
$orm = $orm->withSchema(new Schema([
  'user'    => [
        Schema::MAPPER      => StdMapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => ['id', 'email', 'balance'],
        Schema::TYPECAST    => ['id' => 'int', 'balance' => 'float'],
        Schema::RELATIONS   => [
            'comments' => [
                Relation::TYPE   => Relation::HAS_MANY,
                Relation::TARGET => 'comment',
                Relation::SCHEMA => [
                    Relation::CASCADE   => true,
                    Relation::INNER_KEY => 'id',
                    Relation::OUTER_KEY => 'user_id'
                ],
            ]
        ]
    ],
    'comment' => [
        Schema::MAPPER      => StdMapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'comment',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => ['id', 'user_id', 'message'],
        Schema::TYPECAST    => ['id' => 'int'],
        Schema::RELATIONS   => [],
    ]
]));
```

> More examples can be found in https://github.com/cycle/orm/tree/master/tests/ORM/Classless

## Usage
The only difference in usage of StdClass objects to carry your entity information is the need to identify entities using their roles.

To get access to entity repository:

```php
$userRepository = $orm->getRepository('user');
```

It is also required to create new entities using role specification instead of `new Class()`:

```php
$user = $orm->make('user', [/* fields* /]);
```

> You can freely assign custom repositories and constrains to your entities.

# Example
Example demonstrates database schema declaration and running ORM with manually specified entity schema:

```php
<?php
declare(strict_types=1);

require_once "vendor/autoload.php";

use Cycle\ORM\Factory;
use Cycle\ORM\Mapper\StdMapper;
use Cycle\ORM\ORM;
use Cycle\ORM\Schema;
use Cycle\ORM\Transaction;
use Spiral\Database\Config\DatabaseConfig;
use Spiral\Database\DatabaseManager;
use Spiral\Database\Driver\SQLite\SQLiteDriver;

$dbm = new DatabaseManager(new DatabaseConfig([
    'default'     => 'default',
    'databases'   => [
        'default' => [
            'connection' => 'sqlite',
        ],
    ],
    'connections' => [
        'sqlite' => [
            'driver'     => SQLiteDriver::class,
            'connection' => 'sqlite:database.db',
            'username'   => '',
            'password'   => '',
        ],
    ],
]));

// ensure schema
$users = $dbm->database('default')->table('users')->getSchema();
$users->primary('id');
$users->string('name');
$users->datetime('created_at');
$users->datetime('updated_at');
$users->save();

$orm = new ORM(new Factory($dbm), new Schema([
    'user' => [
        Schema::MAPPER      => StdMapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'users',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => [
            'id'         => 'id',         // property => column_name
            'name'       => 'name',
            'created_at' => 'created_at', 
            'updated_at' => 'updated_at'
        ],
        Schema::TYPECAST    => [
            'id'         => 'int', 
            'created_at' => 'datetime', 
            'updated_at' => 'datetime'
        ],
        Schema::RELATIONS   => [],
    ],
]));

$u = $orm->make('user', [
    'name'       => 'test',
    'created_at' => new DateTimeImmutable(),
    'updated_at' => new DateTimeImmutable(),
]);

(new Transaction($orm))->persist($u)->run();

print_r(
    $orm->getRepository('user')
        ->findAll()
);
```
