# Manually defined Mapping Schema
You can avoid the usage of cycle/annotation extension or any other schema related extension and write the ORM mapping schema manually.

## Custom Schema
You can define custom schema by passing `Schema` object into your ORM instance:

```php
$orm = $orm->withSchema(new Schema([
    // ... schema ...
]));
```

## Define the Entity
To add simple POPO entity mapping:

```php
$orm = $orm->withSchema(new Schema([
   'user' => [
      Schema::ENTITY      => User::class,
      Schema::MAPPER      => Mapper::class,
      Schema::DATABASE    => 'default',
      Schema::TABLE       => 'user',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS     => [
          'id'      => 'id',    // property => column
          'email'   => 'email',
          'balance' => 'balance'
      ],
     Schema::TYPECAST    => [
          'id'         => 'int',
          'balance'    => 'float'
      ],
      Schema::RELATIONS   => []
  ]
]));
```

## Define the relation
To define the relation you must ensure that both entities added to the schema and then populate RELATION section:

```php
$orm = $orm->withSchema(new Schema([
   'user'    => [
      Schema::ENTITY      => User::class,
      Schema::MAPPER      => Mapper::class,
      Schema::DATABASE    => 'default',
      Schema::TABLE       => 'user',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS     => ['id', 'email', 'balance'],
      Schema::RELATIONS   => [
          'profile' => [
              Relation::TYPE   => Relation::HAS_ONE,
              Relation::TARGET => 'profile',
              Relation::SCHEMA => [
                  Relation::CASCADE   => true,
                  Relation::INNER_KEY => 'id',
                  Relation::OUTER_KEY => 'user_id',
              ],
          ]
      ]
  ],
  'profile' => [
      Schema::ENTITY      => Profile::class,
      Schema::MAPPER      => Mapper::class,
      Schema::DATABASE    => 'default',
      Schema::TABLE       => 'profile',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS     => ['id', 'user_id', 'image'],
      Schema::RELATIONS   => []
  ],
]));
```

> Extra fields are omitted.

You can see other relation examples [here](https://github.com/cycle/orm/tree/master/tests).
