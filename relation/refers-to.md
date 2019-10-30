# Refers To
Refers To relation is very similar to Belongs To but must be used in cases when multiple relations can exist to a related entity
(including cyclic relations). For example: a user has many comments, the user refers to the last comment

> The entity will be persisted before the related entity and then updated.

## Definition
Using the annotated extension:

```php
/** @Entity */
class User
{
    // ...

    /** @RefersTo(target="Comment") */
    public $lastComment;

    /** @HasMany(target="Comment") */
    public $comments;

    // ...

    public function addComment(Comment $c)
    {
        $this->lastComment = $c;
        $this->comments->add($c);
    }
}
```

> You must properly handle the cases when the relation is not initialized (`null`)!

By default, the ORM will generate an outer key in the relation object using the related entity's role and outer key (primary key by default) values. As result column and FK will be added to Post entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach. Defaults to `lazy`
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to the primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `SET NULL`
indexCreate | bool   | Create an index on outerKey. Defaults to `true`

> Please note, default `fkAction` is `SET NULL`, the relation is nullable by default.


## Usage
Cycle will automatically save the related entity and link to it (unless `cascade` set to `false`).

```php
$u = new User();
$u->addComment(new Comment("hello world");

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

Simply set the property value to null to remove the entity reference.

```php
$u = new User();
$u->lastComment = null;

$t = new Transaction($orm);
$t->persist($post);
$t->run();
```

### Loading
To access related data call the method `load` of your `Post`'s `Select` object:

```php
$u = $orm->getRepository(User::class)->select()->load('lastComment')->wherePK(1)->fetchOne();
print_r($u->getLastComment());
```

### Filtering
You can filter entity selection using related data, call the method `with` of your entity's `Select` to join the related entity table:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('lastComment')->where('lastComment.approved', true)
    ->fetchAll();

print_r($users);
```

`Select` can automatically join related table on the first `where` condition. The previous example can be rewritten:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->where('lastComment.approved', true)
    ->fetchAll();

print_r($users);
```

### Self References
The RefersTo relation can be used to create self-references.

```php
/** @Entity */
class Category
{
    /** @Column(type="primary") */
    public $id;

    /** @RefersTo(target="Category") */
    public $parent;

    // ...
}
```

To create a tree like that:

```php
$category1 = new Category("A");
$category2 = new Category("A");
$category2->parent = $category1;

$t = new Transaction($orm);
$t->persist($category1);
$t->persist($category2);
$t->run();
```

You can load relations like that on any level (considering memory and performance limitations):

```php
$result = $orm->getRepository(Category::class)
    ->select()
    ->load('parent.parent.parent.parent') // load 4 parent levels
    ->fetchAll();
```

> Make sure that cyclic dependencies are what you need.
