# Many To Many

A relation of type 'Many To Many' provides a more complex connection with the ability to use an intermediate entity for the
connection. The relation requires the `through` option with similar rules as `target`.

'Many To Many' relations are, in fact, two relations combined together. This relation requires an intermediate (pivot)
entity to connect the source and target entities. **Example:** many users have many tags, many posts have many favorites.

The relation provides access to an intermediate object on all the steps, including creation, update and query building.

## Definition

To define a 'Many To Many' relation using the annotated entities' extension, use (attention, make sure to create pivot
entity):

```php
use Cycle\Annotated\Annotation\Relation\ManyToMany;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User
{
    // ...

    #[ManyToMany(target: Tag::class, through: UserTag::class)]
    protected array $tags;
    
    public function getTags(): array
    {
        return $this->tags;
    }
    
    public function addTag(Tag $tag): void
    {
        $this->tags[] = $tag;
    }
    
    public function removeTag(Tag $tag): void
    {
        $this->tags = array_filter($this->tags, static fn(Tag $t) => $t !== $tag);
    }
}
```

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;
}
```

```php
use Cycle\Annotated\Annotation\Relation\ManyToMany;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class Tag
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}
```

By default, ORM will generate FK and indexes in `through` entity using the role and primary keys of the linked objects.
Following values are available for the configuration:

| Option         | Value                        | Comment                                                                                                           |
|----------------|------------------------------|-------------------------------------------------------------------------------------------------------------------|
| load           | lazy/eager                   | Relation load approach. Defaults to `lazy`                                                                        |
| cascade        | bool                         | Automatically save related data with parent entity. Defaults to `false`                                           |
| innerKey       | string                       | Inner key name in source entity. Defaults to a primary key                                                        |
| outerKey       | string                       | Outer key name in target entity. Defaults to a primary key                                                        |
| throughInnerKey | string                       | Key name connected to the innerKey of source entity. Defaults to `{sourceRole}_{innerKey}`                        |
| throughOuterKey | string                       | Key name connected to the outerKey of a related entity. Defaults to `{targetRole}_{outerKey}`                     |
| throughWhere    | array                        | Where conditions applied to `through` entity                                                                       |
| where          | array                        | Where conditions applied to a related entity                                                                      |
| orderBy        | array                        | Additional sorting rules                                                                                          |
| fkCreate       | bool                         | Set to true to automatically create FK on throughInnerKey and throughOuterKey. Defaults to `true`                   |
| fkAction       | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `SET NULL`                                                           |
| fkOnDelete     | CASCADE, NO ACTION, SET NULL | FK onDelete action. It has higher priority than {$fkAction}. Defaults to @see {$fkAction}                         |
| indexCreate    | bool                         | Create index on [throughInnerKey, throughOuterKey]. Defaults to `true`                                              |
| collection     | string                       | Collection type that will contain loaded entities. By defaults uses `Cycle\ORM\Collection\ArrayCollectionFactory`. Read more about [relation collections](collections.md). |

You can keep your pivot entity empty, the only requirement is to have defined a primary key.

> Note, current implementation includes a typo in the pivot table definition, `though` => `through`.

## Usage

To associate two entities using Many To Many relation, use proper way depended on collection type you use. In our 
example we use default collection factory `Cycle\ORM\Collection\ArrayCollectionFactory`:

> Read more about relation collections [here](/docs/en/relation/collections.md).

```php
$user = new User();
$user->setName("Antony");
$user->addTag(new Tag("tag a"));

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

Disassociation will remove the `UserTag` entity, and not the `Tag` entity.

```php
$user->removeTag($tag);
```

### Loading

Use the method `load` of your `Select` object to preload data of related and pivot entities:

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

### Accessing Pivot Entity

If you use `Cycle\ORM\Collection\DoctrineCollectionFactory` for 'Many To Many' relation, you have the ability to
access the pivot entity's data using the `Cycle\ORM\Collection\Pivoted\PivotedCollectionInterface` object. You can do 
that using the `getPivot` method:

```php
use Cycle\Annotated\Annotation\Relation\ManyToMany;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Collection\Pivoted\PivotedCollection;

#[Entity]
class User
{
    // ...
    
    #[ManyToMany(target: Tag::class, through: UserTag::class, collection: 'doctrine')]
    public PivotedCollection $tags;
    
    public function __construct() 
    {
        $this->tags = new PivotedCollection();
    }
}
```

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->fetchAll();

foreach ($users as $user) {
    foreach ($user->tags as $tag) {
         print_r($tag);
         print_r($user->tags->getPivot($tag));
    }
}
```

You can change the values of this entity as they will be persisted with the parent entity. This approach allows you to
easier control the association between parent and related entities.

For example, we can add a new property to our `UserTag`:

```php
use Cycle\Annotated\Annotation\Relation\ManyToMany;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'datetime', default: null)]
    private \DateTimeInterface $created_at;

    public function __construct(\DateTimeInterface $d)
    {
        $this->created_at = $d;
    }
}
```

Now we can assign this entity to the newly created connection:

```php
$u = new User();
$u->setName("Antony");

$tag = new Tag("tag a");

$u->tags->add($tag);
$u->tags->setPivot($tag, new UserTag(new \DateTimeImmutable()));

$t->persist($u);
$t->run();
```

### Filtering

Similar to Has Many the entity query can be filtered using the `with` method:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('tags')
    ->fetchAll();
```

You can filter the entity results using the `where` method on related properties:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.name', 'tag a')
    ->fetchAll();
```

Following SQL will be produced:

```sql
SELECT DISTINCT `user`.`id`   AS `c0`,
                `user`.`name` AS `c1`
FROM `users` AS `user`
         INNER JOIN `user_tags` AS `user_tags_pivot`
                    ON `user_tags_pivot`.`user_id` = `user`.`id`
         INNER JOIN `tags` AS `user_tags`
                    ON `user_tags`.`id` = `user_tags_pivot`.`tag_id`
WHERE `user_tags`.`name` = 'tag a'
```

### Chain Filtering

Pivot entity data is available for filtering as well, you must use the keyword `@` to access it.

```php
$hour = new \DateInterval("PT40M");

$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.created_at', '>', (new \DateTimeImmutable())->sub($hour))
    ->fetchAll();
```

You can also load/filter the relations assigned to the pivot entity.

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.subRelation.value', $value)
    ->fetchAll();
```

> Cross-database Many To Many relations are not supported yet.

## Complex Loading

You can load related data using conditions and sorts applied to the pivot table using the option `load`.

For example, we can have the following entities:

- category (id, title)
- photo (id, url)
- photo_to_category (photo_id, category_id, position)

```php
$categories = $orm->getRepository('category')->select();
```

We can now load categories with photos inside them ordered by `photo_to_category` position using a `WHERE IN` or `JOIN`
query:

```php
$result = $categories->load('photos', [
    'load' => function (\Cycle\ORM\Select\QueryBuilder $q) {
        $q->orderBy('@.@.position'); // @ current relation (photos), @.@ current relation pivot (photo_to_category)
    }
])->fetchAll();
```

The produced SQL:

```sql
SELECT "category"."id"    AS "c0",
       "category"."title" AS "c1"
FROM "categories" AS "category"
```

SQL #2:

```sql
SELECT "l_category_photos_pivot"."id"          AS "c0",
       "l_category_photos_pivot"."position"    AS "c1",
       "l_category_photos_pivot"."photo_id"    AS "c2",
       "l_category_photos_pivot"."category_id" AS "c3",
       "category_photos"."id"                  AS "c4",
       "category_photos"."url"                 AS "c5"
FROM "photos" AS "category_photos"
         INNER JOIN "photo_category_positions" AS "l_category_photos_pivot"
                    ON "l_category_photos_pivot"."photo_id" = "category_photos"."id"
WHERE "l_category_photos_pivot"."category_id" IN (1, 2, 3, 4)
ORDER BY "l_category_photos_pivot"."position" ASC
```

We can force the ORM to use a single query to pull the data (useful for more complex conditions):

```php
$result = $categories->load('photos', [
     'method' => \Cycle\ORM\Select::SINGLE_QUERY,
     'load'   => function (\Cycle\ORM\Select\QueryBuilder $q) {
         $q->orderBy('@.@.position');  // @ current relation (photos), @.@ current relation pivot (photo_to_category)
     }
])->orderBy('id')->fetchAll();
```

SQL:

```sql
SELECT "category"."id"                           AS "c0",
       "category"."title"                        AS "c1",
       "l_l_category_photos_pivot"."id"          AS "c2",
       "l_l_category_photos_pivot"."position"    AS "c3",
       "l_l_category_photos_pivot"."photo_id"    AS "c4",
       "l_l_category_photos_pivot"."category_id" AS "c5",
       "l_category_photos"."id"                  AS "c6",
       "l_category_photos"."url"                 AS "c7"
FROM "categories" AS "category"
         LEFT JOIN "photo_category_positions" AS "l_l_category_photos_pivot"
                   ON "l_l_category_photos_pivot"."category_id" = "category"."id"
         INNER JOIN "photos" AS "l_category_photos"
                    ON "l_category_photos"."id" = "l_l_category_photos_pivot"."photo_id"
ORDER BY "category"."id" ASC, "l_l_category_photos_pivot"."position" ASC
```
