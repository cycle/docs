# Select Entity using Repository

To access the repository associated with a specific entity, use the method `getRepository` of the orm service:

```php
$r = $orm->getRepository(User::class);
```

You can request a repository instance using entity class name or its role name:

```php
$r = $orm->getRepository("user");
```

To find the entity using its primary key:

```php
$entity = $repository->findByPK(1);
```

> Note, the method will return `null` if no entity found.

To find an entity by any of its field(s) use:

```php
$entity = $repository->findOne([
  'name' => 'Antony'
]);
```

> Field names will be automatically mapped to the appropriate column names.

You can use any amount of fields in the method:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => 100
]);
```

If your repository is an instance of `Cycle\ORM\Select\Repository` (SQL), you can also use combined expressions:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => ['>=' => 100]
]);
```

> You can read more about compound expressions [here](/docs/en/database/query-builders.md#selectquery-builder).

To find multiple entities use:

```php
foreach($repository->findAll(['status' => 'active']) as $e) {
  // ...
}
```

## Working with SelectQuery

If the repository is an instance of `Cycle\ORM\Select\Repository` (default SQL repository), you are also able to get
access to low the level method `select`, which gives you the ability to compile more complex queries or preload related
entities:

```php
$result = $repository->select()->where('balance', '>', 1)->load('address')->fetchAll();
```

> It's recommended to avoid usage of the `select` method outside of repository classes, and instead 
> expose [custom](/docs/en/basic/repository.md) find methods.

> You can read more about methods available in select queries [here](/docs/en/database).
