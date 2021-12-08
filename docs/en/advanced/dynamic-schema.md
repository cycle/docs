# Dynamic Mapping Schema
Cycle ORM does not limit the developer's ability to carry entity information using unique classes, it is possible to use specifically crafted model/models to represent entity data in its way. The ORM supports the ability to carry information using StdClass objects out of the box. It is also possible to combine classic annotated entities and StdClasses within one ORM scope.

You can also change the ORM schema at runtime.

## StdMapper
To represent an entity as StdClass you have to change its mapper from the default `Cycle\ORM\Mapper\Mapper` to `Cycle\ORM\Mapper\StdMapper`.
This can be done by either defining your schema using the schema builder or by writing the schema manually.

## Using Schema Builder
We can build our entities using entity definitions:

```php
// Lets create a new entity definition object.
$e = new Entity();
$e->setRole('user');
$e->setMapper(\Cycle\ORM\Mapper\StdMapper::class);


// Declare primary field for the entity.
$field = (new \Cycle\Schema\Definition\Field())
    ->setType('primary')
    ->setColumn('id')
    ->setPrimary(true);

$entity->getFields()->set('id', $field);


// Add entity definition into registry.
$registry = new \Cycle\Schema\Registry($this->dbal);
$registry->register($e)->linkTable($e, 'default', 'user');


// Compile the ORM schema.
$schema = (new \Cycle\Schema\Compiler())->compile($r, []);

print_r($schema);

$orm = $orm->withSchema(new \Cycle\ORM\Schema($schema));
```

> Use SyncTable [generator](/docs/en/advanced/schema-builder.md) to update your database schema.

Relations can also be configured using schema builder:

```php
$entity->getRelations()->set(
    'posts',
    (new \Cycle\Schema\Definition\Relation())->setTarget('post')->setType('hasMany')
);
```

## Manual Schema Definition
You can also define StdClass schema manually using a set of constants exposed by `Cycle\ORM\Schema` and `Cycle\ORM\Relation` classes:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Relation;
use Cycle\ORM\Mapper\StdMapper;

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

> You can freely assign custom repositories and scopes to your entities.

## Example
```php
<?php
declare(strict_types=1);

require_once "vendor/autoload.php";

use Cycle\ORM\Factory;
use Cycle\ORM\Mapper\StdMapper;
use Cycle\ORM\ORM;
use Cycle\ORM\Schema;
use Cycle\ORM\EntityManager;
use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\DatabaseManager;
use Cycle\Database\Driver\SQLite\SQLiteDriver;

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

// automatically migrate database schema if needed (optional)
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

(new EntityManager($orm))->persist($u)->run();

print_r(
    $orm->getRepository('user')
        ->findAll()
);
```
