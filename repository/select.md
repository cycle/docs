# Select Entity using Repository
To access repository associated with specific entity use method `getRepository` of orm service:

```php
$r = $orm->getRepository(User::class);
```

You can request repository instance using entity class name or it's role name:

```php
$r = $orm->getRepository("user");
```

To find the entity using it's primary key:

```php
$entity = $repository->findByPK(1);
```

> Note, method will return `null` if no entity found.

To find entity by any of it's field(s) use:

```php
$entity = $repository->findOne([
  'name' => 'Antony'
]);
```

> Field names will be automatically mapped to appropriate column names.

You can use any amount of fields in request:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => 100
]);
```

If you repository an instance of `Cycle\ORM\Select\Repository` (SQL) you can also use combined expressions:

```php
$entity = $repository->findOne([
  'name'    => 'Antony',
  'balance' => ['>=' => 100]
]);
```

> You can read more about compound expressions [here](https://spiral-framework.com/guide/database-builders).

To find multiple entities use:

```php
foreach($repository->findAll(['status' => 'active']) as $e) {
  // ...
}
```

## Working with SelectQuery
If repository entity is an instance of `Cycle\ORM\Select\Repository` (default SQL repository) you are also able to get access
to low level method `select` which gives you ability to compile more complex queries or pre-load related entities:

```php
$result = $repository->select()->where('balance', '>', 1)->load('address')->fetchAll();
```

> It's recommended to avoid usage of `select` method outside of repository classes and instead expose [custom](repository/custom.md) find methods.

> You can read more about methods available in select queries [here](https://spiral-framework.com/guide/database-builders).
