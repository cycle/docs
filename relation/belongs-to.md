# Belongs To
Belongs To relation defines that entity is owned by a related entity on the exclusive matter. Example: post belongs to the author, comment belongs to post. Most of `belongsTo` relations can be created using the `inverse` option of the declared `hasOne` or `hasMany` relation.

> The entity will be always persisted after related entity.

## Definition
To define Belongs To relation using annotated entities extension use:

```php
/** @Entity */
class Post
{
    // ...

    /** @BelongsTo(target = "User") */
    protected $user;
}
```

> You must properly handle the cases when the relation is not initialized (`null`)!

By default, ORM will generate an outer key in relation object using related entity role and outer key (primary key by default) values. As result column and FK will be added to Post entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach (default `lazy`)
cascade     | bool   | Automatically save related data with source entity, defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in source entity, defaults to `{relationName}_{outerKey}`
outerKey    | string | Outer key in the related entity, by default primary key
fkCreate    | bool   | Set to true to automatically create FK on innerKey, defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `CASCADE`
indexCreate | bool   | Create an index on innerKey, defaults to `true`

# Usage
Cycle will automatically save the related entity (unless `cascade` set to `false`).

```php
$post = new Post();
$post->setUser(new User("Antony"));

$t = new Transaction($orm);
$t->persist($post);
$t->run();
```

You can only de-associate related entity if relation set as `nullable`, in other scenarios you will get an integrity exception:

```php
$post = $orm->getRepository(Post::class)->findOne();

$post->setUser(null);

$t = new Transaction($orm);
$t->persist($post);
$t->run();
```

## Loading
To access related data simply call the method `load` of your `Post` `Select` object:

```php
$p = $orm->getRepository(Post::class)->select()->load('user')->wherePK(1)->fetchOne();
print_r($p->getAddress());
```

## Filtering
You can filter entity selection using related data, call method `with` of your entity `Select` to join related entity table:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->with('user')->where('user.status', 'active')
    ->fetchAll();

print_r($posts);
```

Cycle `Select` can automatically join related table on first `where` condition, previous example can be rewritten:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->where('user.status', 'active')
    ->fetchAll();

print_r($posts);
```
