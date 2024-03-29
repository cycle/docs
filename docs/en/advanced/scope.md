# Scope

Every query generated for the entity selection (directly or via relation) might pass through the defined scope. Scopes
are used to define global query limits or/and filter entity by one of its relations.

<img width="816" alt="Scope schema" src="https://user-images.githubusercontent.com/773481/144860939-f18d4fb0-083a-44fa-9691-73c95ecd069c.png">

In some cases, you can disable scope usage on root query to get access to unfiltered entities.
Use `$select->scope(null)` to do that.

## Example

A simple example can demonstrate how to only select entities which are not marked as `deleted`:

```php
use Cycle\ORM\Select;

class NotDeletedScope implements Select\ScopeInterface
{
    public function apply(Select\QueryBuilder $query)
    {
        $query->where('deleted_at', '=', null);
    }
}
```

Scope can be assigned to any entity via schema, in case of `annotated` extension:

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity(scope: NotDeletedScope::class)]
class User
{
    // ...
}
```
