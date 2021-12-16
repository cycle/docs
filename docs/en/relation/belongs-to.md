# Belongs To

**Belongs To** relation defines that an entity is owned by a related entity on the exclusive matter. **Example:** a post
belongs to an author, a comment belongs a post. Most `belongsTo` relations can be created using the `inverse` option of
the declared `hasOne` or `hasMany` relation.

> The entity will always be persisted after its related entity.

## Definition

To define Belongs To relation using annotated entities extension use:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\BelongsTo;

#[Entity]
class Post
{
    // ...

    #[BelongsTo(target: User::class)]
    private User $user;
}
```

> You must properly handle the cases when the relation is not initialized (`null`)!

By default, the ORM will generate an outer key in the relation object using the related entity's role and outer key (
primary key by default) values. As result column and FK will be added to Post entity on `user_id` column. 

Option      | Value                        | Comment
---         |------------------------------| ----
load        | lazy/eager                   | Relation load approach. Defaults to `lazy`
cascade     | bool                         | Automatically save related data with source entity. Defaults to `true`
nullable    | bool                         | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string,array                 | Inner key in source entity. Defaults to `{relationName}_{outerKey}`
outerKey    | string,array                 | Outer key in the related entity. Defaults to the primary key
fkCreate    | bool                         | Set to true to automatically create FK on innerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
fkOnDelete  | CASCADE, NO ACTION, SET NULL | FK onDelete action. It has higher priority than {$fkAction}. Defaults to @see {$fkAction}
indexCreate | bool                         | Create an index on innerKey. Defaults to `true`

> Since Cycle ORM v2.x `innerKey` and `outerKey` can contain [composite key](/docs/en/advanced/composite-pk.md).

## Usage

ORM will automatically save the related entity (unless `cascade` set to `false`).

```php
$post = new Post();
$post->setUser(new User("Antony"));

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->run();
```

You can only de-associate the related entity if the relation is defined as `nullable`. In other scenarios you will get
an integrity exception:

```php
$post = $orm->getRepository(Post::class)->findOne();

$post->setUser(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->run();
```

### Loading

To access related data simply call the method `load` of your `Post` `Select` object:

```php
$post = $orm->getRepository(Post::class)->select()->load('user')->wherePK(1)->fetchOne();
print_r($post->getAddress());
```

### Filtering

You can filter entity selection using related data, call the method `with` of your entity's `Select` to join the related
entity table:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->with('user')->where('user.status', 'active')
    ->fetchAll();

print_r($posts);
```

`Select` can automatically join related tables on the first `where` condition. The previous example can be rewritten:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->where('user.status', 'active')
    ->fetchAll();

print_r($posts);
```
