# Query Builder Basics
ORM provides control over generated SQL statements and functionaity to set custom conditions and parameters.

## Accessing Query Builder
You can get access to query builder manually by constructing instance of `Select` object or by using `select()` method of default repository class.

```php
$select = $orm->getRepository(User::class)->select();

// ...
```

The recommended approach is to declare model select statements within entity repository:

```php
class UserRepository extends Repository
{
    public function findActive(): Select
    {
        return $this->select()->where('status', 'active');
    }
}
```

## Simple conditions
You can set any condition on obtained query builder using method `where`:

```php
$select->where('status', 'active');
```

By default such condition will generate statement like `'status' = "active"` (value will be passed as part of prepared statement).

To specify custom operator declare function with 3 arguments:

```php
$select->where('balance', '>', 100);
```

For `between` and `not between` conditions you can use notation with 4 arguments:

```php
$select->where('id', 'between', 10, 20);
```

You can use `orWhere` and `andWhere` (idential to `where`) to declare more complex conditions:

```php
$select->where('balance', '<', 100)->orWhere('status', 'blocked');
```

> Read more of complex conditions in the next article.

## Short Notation
You can also specify some conditions using array notation:

```php
$select->where([
    'id'     => ['in' => new Parameter([1, 2, 3])],
    'status' => ['like' => 'active']
]);
```

Such declaration is identical to:

```php
$select->where(function(QueryBuilder $select) {
    $select->where('id', 'in', new Parameter([1, 2, 3]));
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
By default, any passed value will be converted into `Parameter` object internally. Hovewer, you must clearly use `Parameter` while specifying array values:

```php
$select->where('id', 'in', new Parameter([1,2,3]));
```

In order cases parameters can be used to specify value after building the query:

```php
$select->where('id', $id = new Parameter(null));

$id->setValue(10);

print_r($select->fetchAll());
```

## Sorting and pagination
Use available methods `offset`, `limit` and `orderBy` to paginate or sort your entities:

```php
$select->orderBy('id', 'DESC')->limit(1);
```
