# Manually defined Mapping Schema

You can avoid the usage of the `cycle/annotation` extension or any other schema related extension and write the ORM
mapping schema manually.

## Custom Schema

You can define a custom schema by passing a `Schema` object into your ORM instance:

```php
$orm = $orm->with(schema: new \Cycle\ORM\Schema([
    // ... schema ...
]));
```

## Define the Entity

To add a simple POPO entity mapping:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Mapper\Mapper;

$orm = $orm->with(schema: new Schema([
   'user' => [
      Schema::ENTITY => User::class,
      Schema::MAPPER => Mapper::class,
      Schema::DATABASE => 'default',
      Schema::TABLE => 'user',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS => [
          // property => column
          'id' => 'id',
          'email' => 'email',
          'balance' => 'balance'
      ],
     Schema::TYPECAST => [
          'id' => 'int',
          'balance' => 'float'
      ],
      Schema::RELATIONS   => []
  ]
]));
```

## Define the relation

To define the relation you must ensure that both entities are added to the schema and then populate RELATION section:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Relation;
use Cycle\ORM\Mapper\Mapper;

$orm = $orm->with(schema: new Schema([
   'user' => [
      Schema::ENTITY => User::class,
      Schema::MAPPER => Mapper::class,
      Schema::DATABASE => 'default',
      Schema::TABLE => 'user',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS => ['id', 'email', 'balance'],
      Schema::RELATIONS => [
          'profile' => [
              Relation::TYPE => Relation::HAS_ONE,
              Relation::TARGET => 'profile',
              Relation::SCHEMA => [
                  Relation::CASCADE  => true,
                  Relation::INNER_KEY => 'id',
                  Relation::OUTER_KEY => 'user_id',
              ],
          ]
      ]
  ],
  'profile' => [
      Schema::ENTITY => Profile::class,
      Schema::MAPPER => Mapper::class,
      Schema::DATABASE => 'default',
      Schema::TABLE => 'profile',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS => ['id', 'user_id', 'image'],
      Schema::RELATIONS => []
  ],
]));
```

> Extra fields are omitted.

You can see other relation examples [here](https://github.com/cycle/orm/tree/master/tests).
