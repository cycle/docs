# Query Builder Basics

The ORM provides control over generated SQL statements and functionality to set custom conditions and parameters.

## Accessing Query Builder

You can get access to the query builder manually by constructing an instance of the `Select` object, or by using
the `select()` method of the default repository class.

```php
$select = $orm->getRepository(User::class)->select();

// ...
```

The recommended approach is to declare select statements within an entity's repository:

```php
use Cycle\ORM\Select;

class UserRepository extends Select\Repository
{
    public function findActive(): Select
    {
        return $this->select()->where('status', 'active');
    }
}
```

## Simple conditions

You can set any condition on the obtained query builder using the method `where`:

```php
$select->where('status', 'active');
```

By default, such condition will generate statement like `'status' = "active"` (value will be passed as part of the
prepared statement).

To specify a custom operator call the function with 3 arguments:

```php
$select->where('balance', '>', 100);
```

For `between` and `not between` conditions you can use notation with 4 arguments:

```php
$select->where('id', 'between', 10, 20);
```

You can use `orWhere` and `andWhere` (identical to `where`) to declare more complex conditions:

```php
$select->where('balance', '<', 100)->orWhere('status', 'blocked');
```

> Read more of complex conditions in the next article.

## Short Notation

You can also specify conditions using array notation:

```php

$select->where([
    'id'     => ['in' => new \Cycle\Database\Injection\Parameter([1, 2, 3])],
    'status' => ['like' => 'active']
]);
```

This declaration is identical to:

```php
$select->where(function(\Cycle\ORM\Select\QueryBuilder $select) {
    $select->where('id', 'in', new \Cycle\Database\Injection\Parameter([1, 2, 3]));
    $select->where('status', 'like', 'active');
});
```

Array notation can be used to declare multiple conditions on one field:

```php
$select->where([
    'id' => ['>' => 10, '<' => 100]
]);
```

## Using Parameters

By default, any passed value will be converted into `Parameter` objects internally. However, you must explicitly
use `Parameter` while specifying array values:

```php
$select->where('id', 'in', new \Cycle\Database\Injection\Parameter([1,2,3]));
```

Parameters can be used to specify a value after building the query:

```php
$select->where('id', $id = new \Cycle\Database\Injection\Parameter(null));

$id->setValue(10);

print_r($select->fetchAll());
```

## Sorting and pagination

Use the methods `offset`, `limit` and `orderBy` to paginate or sort your entities:

```php
$select->orderBy('id', 'DESC')->limit(1);
```
