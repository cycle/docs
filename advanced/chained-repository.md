# Chained Repository
The Cycle default Repository implementation provides the ability to be easility clone the resitory object in order to affect the base query.

## Custom Scopes
To implement the ability to create custom query scope you must implement method to clone the respotitory and customize base select query:


```php
class UserRepository extends Repository 
{
    public function withActive(): self
    {
        $r = clone $this;
        $r->select->where('status', 'active');
        
        return $r;
    }
}
```

You can use this method now to change the repository behaviour:

```php
$r = $orm->getRepository(User::class);

print_r($r->withActive()->findAll());
```

> You can chain as many scope methods as you want, make sure to keep the repository state immutable.

## Disable the Constrain 
If you use entity constrain (for example soft-deleted) you can alter your underlying select query to disable it in specific cases:

```php
class UserRepository extends Repository 
{
    public function withDeleted(): self
    {
        $r = clone $this;
        $r->select->constrain(null);
        
        return $r;
    }
}
```

