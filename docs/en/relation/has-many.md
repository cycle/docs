# Has Many

The Has Many relations defines that an entity exclusively owns multiple other entities in a form of parent-children.

## Definition

To define a Has Many relation using the annotated entities extension, use:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class)]
    private array $posts;
}
```

To use a newly created entity you must define the collection to store related entities. Do it in your constructor:

```php
use Doctrine\Common\Collections\ArrayCollection;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class)]
    protected ArrayCollection $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    // ...

    public function getPosts(): ArrayCollection
    {
        return $this->posts;
    }
}
```

By default, the ORM will generate an outer key in the relation object using the parent entity's role and inner key (
primary key by default) values. As result column and FK will be added to Post entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach. Defaults to `lazy`
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to the primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
where       | array  | Additional where condition to be applied for the relation. Defaults to none
orderBy     | array  | Additional sorting rules. Defaults to none
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
fkOnDelete  | CASCADE, NO ACTION, SET NULL | FK onDelete action. It has higher priority than {$fkAction}. Defaults to @see {$fkAction}
indexCreate | bool   | Create an index on outerKey. Defaults to `true`
collection  | string | Collection type that will contain loaded entities. By defaults uses `Cycle\ORM\Collection\ArrayCollectionFactory`

## Usage

To add the child object to the collection, use the collection method `add`:

```php
$u = new User();
$u->getPosts()->add(new Post("test post"));
```

The related object(s) can be immediately saved into the database by persisting the parent entity:

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u);
$manager->run();
```

To delete a previously associated object, call the `remove` or `removeElement` methods of the collection:

```php
$post = $u->getPosts()->get(0);
$u->getPosts()->removeElement(post);
```

The child object will be removed during the persist operation.

> Set the relation option `nullable` as true to nullify the outer key instead of entity removal.

### Loading

To access related data simply call the method `load` of your `User`'s `Select` object:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('posts')
    ->wherePK(1)
    ->fetchAll();

foreach ($users as $u) {
    print_r($u->getPosts());
}
```

Please note, by default ORM will load HasMany related entities using an external query (`WHERE IN`).

### Filtering

You can filter entity selection using related data. Call the method `with` of your entity's `Select` to join the related
entity table:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')->where('posts.published', true)
    ->fetchAll();

print_r($users);
```

> Make sure to call `distinct` since the multi-row table will be joined to your query.

`Select` can automatically join related tables on the first `where` condition. The previous example can be rewritten:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('posts.published', true)
    ->fetchAll();

print_r($users);
```

### Load filtered

Another option available for HasMany relation is to pre-filter related data on the database level. To do that, use
the `where` option of the relation, or the`load` method. For example, we can load all users with at least one post and
pre-load only published posts:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')
    ->load('posts', ['where' => ['published' => true]])
    ->fetchAll();
```

Another option is to use the `with` selection to drive the data for the pre-loaded entities. You can point your `load`
method to use `with` filtered relation data via `using` flag:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts', ['as' => 'published_posts'])->where('posts.published', true)
    ->load('posts', ['using' => 'published_posts'])
    ->fetchAll();
```

The given approach will produce only one SQL query.

```sql
SELECT DISTINCT `user`.`id`                   AS `c0`,
                `user`.`name`                 AS `c1`,
                `published_posts`.`id`        AS `c2`,
                `published_posts`.`title`     AS `c3`,
                `published_posts`.`published` AS `c4`,
                `published_posts`.`user_id`   AS `c5`,
FROM `spiral_users` AS `user`
         INNER JOIN `spiral_posts` AS `published_posts`
                    ON `published_posts`.`user_id` = `user`.`id`
WHERE `published_posts`.`published` = true
```

You can also pre-set the conditions in the relation definition:

```php
use Doctrine\Common\Collections\ArrayCollection;
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class, where: ['published' => true])]
    protected ArrayCollection $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    // ...

    public function getPosts(): ArrayCollection
    {
        return $this->posts;
    }
}
```

### Load sorted

You can specify the sort order of the loaded data. The new rule will override the default if specified.

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')
    ->load('posts', ['where' => ['published' => true], ['orderBy' => ['published_at' => 'DESC']]])
    ->fetchAll();
```

The default sort order can also be preset in the relationship definition:

```php
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class, orderBy: ['published' => 'DESC'])]
    protected $posts;
}
```
