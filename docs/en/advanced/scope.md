# Scope

Every query generated for the entity selection (directly or via relation) might pass through the defined scope.
Scopes are used to define global query limits or/and filter entity by one of its relations.

<img width="816" alt="Screenshot_76" src="https://user-images.githubusercontent.com/796136/59182959-ae1ac280-8b73-11e9-819f-d3966ef691a6.png">

In some cases, you can disable scope usage on root query to get access to unfiltered entities. Use `$select->scope(null)` to do that.

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
