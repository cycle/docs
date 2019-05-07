# Referes To
Refers To relation is very similar to Belongs To but must be used in cases when multiple relations can exist to related entity
(including cyclic relations). Example: user has many comments, user refers to last comment


## Definition
Using annotated extension:

```php
/** @entity */
class User
{
    // ...
    
    /** @refersTo(target="Comment") */
    public $lastComment;

    /** @hasMany(target="Comment") */
    public $comments;

    // ...

    public function addComment(Comment $c)
    {
        $this->lastComment = $c;
        $this->comments->add($c);
    }
}
```

> You must properly handle the cases when relation is not initialized (`null`)!

By default, ORM will generate outer key in relation object using related entity role and outer key (primary key by default) values. As result column and FK will be added to Post entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}`
fkCreate    | bool   | Set to true to automatically create FK on outerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `SET NULL`  
indexCreate | bool   | Create index on outerKey, defaults to `true`

> Please note, default `fkAction` is `SET NULL`, relation is nullable by default.


# Usage
Cycle will automatically save the related entity and link to it (unless `cascade` set to `false`).

```php
$u = new User();
$u->addComment(new Comment("hello world");

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

Simply set the properly value to null to remove the entity reference.

```php
$u = new User();
$u->lastComment = null;

$t = new Transaction($orm);
$t->persist($post);
$t->run();
```

## Loading
To access related data call the method `load` of your `Post` `Select` object:

```php
$u = $orm->getRepository(User::class)->select()->load('lastComment')->wherePK(1)->fetchOne();
print_r($u->getLastComment());
```

## Filtering
You can filter entity selection using related data, call method `with` of your entity `Select` to join related entity table:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('lastComment')->where('lastComment.approved', true)
    ->fetchAll();
    
print_r($users);
```

Cycle `Select` can automatically join related table on first `where` condition, previous example can be rewritten:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->where('lastComment.approved', true)
    ->fetchAll();
    
print_r($users);
```

## Self References
The RefersTo relation can be used to create self references.

```php
/** @entity */
class Category 
{
    /** @column(type=primary) */
    public $id;
    
    /** @refersTo(target=Category) */
    public $parent;
    
    // ...
}
```

To create tree like that:

```php
$category1 = new Category("A");
$category2 = new Category("A");
$category2->parent = $category1;

$t = new Transaction($orm);
$t->persist($category1);
$t->persist($category2);
$t->run();
```

You can load relations like that on any level (concidering memory and performance limitations):

```php
$result = $orm->getRepository(Category::class)
    ->select()
    ->load('parent.parent.parent.parent') // load 4 parent levels
    ->fetchAll();
```

> Make sure that cyclic dependencies is what you need.
