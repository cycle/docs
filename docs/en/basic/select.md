# Select Entity

Cycle ORM provides multiple options to select entity data from the database. The most common and recommended method to
use the associated entity repository.

## Using Repository

To access the repository associated with a specific entity use the method `getRepository` of the orm service:

```php
$repository = $orm->getRepository(User::class);
```

You can request a repository instance using the entity's class name or its role name:

```php
$repository = $orm->getRepository("user");
```

The Repository provides multiple methods to select the entity.

To find the entity using its primary key:

```php
$entity = $repository->findByPK(1);
```

> Note, the method will return `null` if no entity is found.

To find an entity by any of its field(s) use:

```php
$entity = $repository->findOne([
  'name' => 'Antony'
]);
```

> Field names will be automatically mapped to appropriate column names.

You can use any amount of fields in a request:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => 100
]);
```

If your repository is an instance of `Cycle\ORM\Select\Repository` (SQL) you can also use combined expressions:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => ['>=' => 100]
]);
```

> You can read more about compound expressions [here](/docs/en/query-builder/complex.md).

To find multiple entities use:

```php
foreach($repository->findAll(['status' => 'active']) as $e) {
  // ...
}
```

## Working with SelectQuery

If the repository entity is an instance of `Cycle\ORM\Select\Repository` (default SQL repository) you can also access
the low level method `select`, which grants you the ability to construct more complex queries or pre-load related
entities:

```php
$result = $repository->select()->where('balance', '>', 1)->load('address')->fetchAll();
```

> It's recommended to avoid usage of `select` method outside of repository classes and instead expose 
> [custom](/docs/en/basic/repository.md) find methods.

> You can read more about the methods available in select queries [here](/docs/en/database/query-builders.md).

## The repository Scope

Please note, in Cycle ORM the Repository object is only responsible for entity retrieval. All persist operations must be
handled by entity manager, entity mappers and command chains.
