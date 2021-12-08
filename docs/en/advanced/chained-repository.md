# Chained Repository

The Cycle ORM default Repository implementation provides the ability to be easily cloned without affecting the base
query for the original instance.

## Custom Scopes

To implement the ability to create a custom query scope you must implement a method to clone the repository and
customize the base select query:

```php
class UserRepository extends \Cycle\ORM\Select\Repository
{
    public function withActive(): self
    {
        $repository = clone $this;
        $repository->select->where('status', 'active');

        return $repository;
    }
}
```

You can now use this method to change the repository behaviour:

```php
$repository = $orm->getRepository(User::class);

print_r($repository->withActive()->findAll());
```

> You can chain as many scope methods as you want, make sure to keep the repository state immutable.

## Disable the scope

If you use entity scope (for example soft-deleted) you can alter your underlying select query to disable it in specific
cases:

```php
class UserRepository extends \Cycle\ORM\Select\Repository
{
    public function withDeleted(): self
    {
        $repository = clone $this;
        $repository->select->scope(null);

        return $repository;
    }
}
```
