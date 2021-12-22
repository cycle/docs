# Refers To

The RefersTo relation is similar to the BelongsTo relation, but must be used to establish **multiple relations** to the
same entity (or in case of a **cyclic** relation). The most common example is the ability to store the last comment posted
by the user.

> The entity will be persisted before the related entity and then updated.

## Definition

Using the annotated extension:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\RefersTo;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[RefersTo(target: Comment::class)]
    private ?Comment $lastComment;

    #[HasMany(target: Comment::class)]
    public array $comments;

    // ...

    public function addComment(Comment $c)
    {
        $this->lastComment = $c;
        $this->comments[] = $c;
    }
    
    public function removeLastComment(): void
    {
        $this->lastComment = null;
    }
    
    public function getLastComment(): void
    {
        return $this->lastComment;
    }
}
```

> You must properly handle the cases when the relation is not initialized (`null`)!

By default, the ORM will generate an outer key in the relation object using the related entity's role and outer key (
primary key by default) values. As result column and FK will be added to Post entity on `user_id` column.

| Option      | Value                        | Comment                                                                                   |
|-------------|------------------------------|-------------------------------------------------------------------------------------------|
| load        | lazy/eager                   | Relation load approach. Defaults to `lazy`                                                |
| cascade     | bool                         | Automatically save related data with parent entity. Defaults to `true`                    |
| nullable    | bool                         | Defines if the relation can be nullable (child can have no parent). Defaults to `false`   |
| innerKey    | string                       | Inner key in parent entity. Defaults to the primary key                                   |
| outerKey    | string                       | Outer key name. Defaults to `{parentRole}_{innerKey}`                                     |
| fkCreate    | bool                         | Set to true to automatically create FK on outerKey. Defaults to `true`                    |
| fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `SET NULL`                                   |
| fkOnDelete  | CASCADE, NO ACTION, SET NULL | FK onDelete action. It has higher priority than {$fkAction}. Defaults to @see {$fkAction} |
| indexCreate | bool                         | Create an index on outerKey. Defaults to `true`                                           |

> Please note, default `fkAction` is `SET NULL`, the relation is nullable by default.

## Usage

Cycle ORM will automatically save the related entity and link to it (unless `cascade` set to `false`).

```php
$user = new User();
$user->addComment(new Comment("hello world"));

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

Simply set the property value to null to remove the entity reference.

```php
$user = new User();
$user->removeLastComment();;

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

### Loading

To access related data call the method `load` of your `Post`'s `Select` object:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('lastComment')
    ->wherePK(1)
    ->fetchOne();

print_r($user->getLastComment());
```

### Filtering

You can filter entity selection using related data, call the method `with` of your entity's `Select` to join the related
entity table:

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
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\RefersTo;

#[Entity]
class Category
{
     #[Column(type: 'primary')]
    public int $id;

     #[RefersTo(target: Category::class)]
    public ?Category $parent;

    // ...
}
```

To create a tree like that:

```php
$category1 = new Category("A");
$category2 = new Category("A");
$category2->parent = $category1;

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($category1);
$manager->persist($category2);
$manager->run();
```

You can load relations like that on any level (considering memory and performance limitations):

```php
$result = $orm->getRepository(Category::class)
    ->select()
    ->load('parent.parent.parent.parent') // load 4 parent levels
    ->fetchAll();
```

> Make sure that cyclic dependencies are what you need.
