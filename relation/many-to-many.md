# Many To Many
Many to Many relation is, in fact, two relations combined together. This relation requires intermediate (pivot) entity to connect the source and target entities. Examples: many users have many tags, many posts have many favorites.

The relation provides access to an intermediate object on all the steps including creation, update and query building.

## Definition
To define Many To Many relation using annotated enties extension use (attention, make sure to create pivot entity):

```php
/** @Entity */ 
class User 
{
    // ...
    
    /** @ManyToMany(target = "Tag", though = "UserTag") */
    protected $tags;
}
```

In order to use a newly created entity, you must define the collection to store related entities. The collection must be an instance of
`Cycle\ORM\Relation\Pivoted\PivotedCollection`. Do it in your constructor:

```php
use use Cycle\ORM\Relation\Pivoted\PivotedCollection;

/** @Entity */ 
class User 
{
    // ...
    
   /** @ManyToMany(target = "Tag", though = "UserTag") */
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

```php
/** @Entity */
class UserTag
{
    /** @Column(type=primary) */
    private $id;
}
```
 
```php
/** @Entity */
class Tag
{
    /** @Column(type=primary) */
    private $id;

    /** @Column(type=string) */
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }
}
```

By default, ORM will generate FK and indexes in `though` entity using role and primary keys of linked objects. Following values are available for the configuration:

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach (default `lazy`)
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
innerKey    | string | Inner key name in source entity, default to a primary key
outerKey    | string | Outer key name in target entity, default to a primary key
thoughInnerKey | string | Key name connected to the innerKey of source entity defaults to `{sourceRole}_{innerKey}` 
thoughOuterKey | string | Key name connected to the outerKey of a related entity defaults to `{targetRole}_{outerKey}` 
thoughWhere | array | Where conditions applied to `though` entity
where       | array | Where conditions applied to a related entity
fkCreate    | bool   | Set to true to automatically create FK on thoughInnerKey and thoughOuterKey, defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `SET NULL`  
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`

> You can keep your pivot entity empty, the only requirement is to have defined a primary key.
 
## Usage
To associte two entities using Many To Many relation use method `add` of pivot collection:

```php
$u = new User();
$u->setName("Antony");

$u->getTags()->add(new Tag("tag a"));

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

To remove association to the object using `remove` or `removeElement` methods. Disassociation will remove `UserTag` not `Tag` entity.

```php
$u->getTags()->removeElement($tag);
```

## Loading
Use method `load` of your `Select` object to pre-load data of related and pivot entities:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->fetchAll();
```

Once loaded, you can access the related entity data using the collection:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->fetchAll();
    
foreach ($users as $u) {
    print_r($u->getTags()->toArray());
}
```

## Accessing Pivot Entity
Many To Many relation provides you ability to access the pivot entity data using the `PivotedCollection` object. You can do that
using `getPivot` method:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->fetchAll();
    
foreach ($users as $u) {
    foreach ($u->getTags() as $t) {
         print_r($t);
         print_r($u->getTags()->getPivot($t));
    }
}
```

You can change the values of this entity as they will be persisted with the parent entity. This approach allows you to easier
control the association between parent and related entities.

For example, we can add new properly to our `UserTag`:

```php
/** @Entity */
class UserTag
{
    /** @Column(type=primary) */
    private $id;

    /** @Column(type=datetime, default = null) */
    private $created_at;

    public function __construct(\DateTimeInterface $d)
    {
        $this->created_at = $d;
    }
}
```

Now we can assign this entity to newly created connection:

```php
$u = new User();
$u->setName("Antony");
        
$tag = new Tag("tag a");

$u->tags->add($tag);
$u->tags->setPivot($tag, new UserTag(new \DateTimeImmutable()));
    
$t->persist($u);
$t->run();
```

## Filtering
Similar to Has Many the entity query can be filtered using `with` method:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('tags')
    ->fetchAll();
```

You can filter the entity results using `where` method on related properties:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.name', 'tag a')
    ->fetchAll();
```

Following SQL will be produced:

```sql 
SELECT DISTINCT
`user`.`id` AS `c0`, `user`.`name` AS `c1`
FROM `users` AS `user`
INNER JOIN `user_tags` AS `user_tags_pivot`
    ON `user_tags_pivot`.`user_id` = `user`.`id`
INNER JOIN `tags` AS `user_tags`
    ON `user_tags`.`id` = `user_tags_pivot`.`tag_id`
WHERE `user_tags`.`name` = 'tag a'
```

## Chain Filtering
Pivot entity data is available for filtering as well, you must use the keyword `@` to access it.

```php
$hour = new \DateInterval("PT40M");

$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.created_at', '>', (new \DateTimeImmutable())->sub($hour))
    ->fetchAll();
```

You can also load/filter the relations assigned to pivot entity.

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.subRelation.value', $value)
    ->fetchAll();
```

> Cross database Many To Many is not supported yet.
