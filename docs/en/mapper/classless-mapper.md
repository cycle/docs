# Classless Mapper

Cycle ORM provides the ability to create proxy classes on the fly by using `Cycle\ORM\Mapper\ClasslessMapper`. There is
no needs to create entity classes.

> The proxy entity performs "lazy loading" for properties with relations. It means that relation data will 
> only be loaded when you actually access them.

## Define the Entity schema

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Mapper\ClasslessMapper;

$orm = $orm->with(schema: new Schema([
   'user' => [
      Schema::MAPPER => ClasslessMapper::class,
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
$user = $this->orm->make('user'); // \Cycle\ORM\ClasslessProxy\Classless user 0 Cycle ORM Proxy object

$user->email = 'user@site.com';
$user->balance = 1500;

(new EntityManager($orm))->persist($user)->run();
```

## Fetching entity data

```php
$repository = $orm->getRepository('user');

$user = $repository->findByPK(1);

var_dump($user); // \Cycle\ORM\ClasslessProxy\Classless user 0 Cycle ORM Proxy object
```

> Note: All proxy entities implement `\Cycle\ORM\EntityProxyInterface` interface.
