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
