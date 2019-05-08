# Many To Many
Many to Many relation is in fact two relations combined together. This relation require intermediate (pivot) entity to connect source and target entities. Examples: many users has many tags, many posts has many favorites.

The relation provides the access to intermediate object on all the steps including creation, update and query building.

## Definition
To define Has Many relation using annotated enties extension use (attention, make sure to create pivot entity):

```php
/** @entity */ 
class User 
{
    // ...
    
    /** @manyToMany(target = "Tag", though = "UserTag") */
    protected $tags;
}
```

In order to use newly created entity you must define the collection to store related entities. The collection must be instance of
`Cycle\ORM\Relation\Pivoted\PivotedCollection`. Do it in your constructor:

```php
use use Cycle\ORM\Relation\Pivoted\PivotedCollection;

/** @entity */ 
class User 
{
    // ...
    
   /** @manyToMany(target = "Tag", though = "UserTag") */
    protected $tags;

    public function __construct()
    {
        $this->tags = new PivotedCollection();
    }
       
    // ...
    
    public function getTags()
    {
        return $this->tags;
    }
}
```

By default ORM will generate FK and indexes in `though` entity using role and primary keys of linked objects. Following values are available for the configuration:


Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (pivot entity can exists without parent(s)), defaults to `true`
innerKey    | string | Inner key name in source entity, default to primary key
outerKey    | string | Outer key name in target entity, default to primary key
thoughInnerKey | string | Key name connected to the innerKey of source entity, defaults to `{sourceRole}_{innerKey}` 
thoughOuterKey | string | Key name connected to the outerKey of related entity, defaults to `{targetRole}_{outerKey}` 
thoughWhere | array | Where conditions applied to `though` entity
where       | array | Where conditions applied to related entity
fkCreate    | bool   | Set to true to automatically create FK on thoughInnerKey and thoughOuterKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `SET NULL`  
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`
