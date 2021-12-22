# Std Mapper

Cycle ORM provides the ability to carry data over the StdClass objects by using `Cycle\ORM\Mapper\StdMapper`
with `\Cycle\ORM\Reference\Promise` objects for relations with lazy loading.

## Define the Entity schema

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Mapper\StdMapper;

$orm = $orm->with(schema: new Schema([
   'user' => [
      Schema::MAPPER => StdMapper::class,
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

## Persisting entity data

```php
$user = $this->orm->make('user'); // stdClass object

$user->email = 'user@site.com';
$user->balance = 1500;

(new EntityManager($orm))->persist($user)->run();
```

> Note: If you want to create entity object manually like `$user = new \stdClass()`, you have to add this objects to 
> ORM Heap, otherwise Entity Manager won't persist the entity.

## Fetching entity data

```php
$repository = $orm->getRepository('user');

$user = $repository->findByPK(1);

var_dump($user); // stdClass object
```
