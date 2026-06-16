# Refers To

The **RefersTo** relation is similar to BelongsTo but designed for specific use cases:

- **Multiple relations to the same entity** (e.g., `createdBy` and `modifiedBy` both pointing to `User`)
- **Cyclic/self-referencing relations** (e.g., `Category` → `parentCategory`)

> The entity is persisted first, then the related entity is saved, and finally the entity is updated with the relation
> reference.

## Definition

To define a RefersTo relation using the annotated entities extension:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\RefersTo;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $username;

    #[RefersTo(target: Comment::class)]
    private ?Comment $lastComment = null;

    #[HasMany(target: Comment::class)]
    private array $comments = [];

    public function addComment(Comment $comment): void
    {
        $this->lastComment = $comment;
        $this->comments[] = $comment;
    }

    public function getLastComment(): ?Comment
    {
        return $this->lastComment;
    }

    public function removeLastComment(): void
    {
        $this->lastComment = null;
    }
}
```

### Attribute Specification

| Parameter   | Type          | Default   | Description                                                                                 |
|-------------|---------------|-----------|---------------------------------------------------------------------------------------------|
| target      | string        | -         | **Required**. Target entity role or class name                                              |
| load        | string        | 'lazy'    | Loading strategy: 'lazy' or 'eager'                                                         |
| cascade     | bool          | true      | Automatically save related entity with source entity                                        |
| nullable    | bool          | false     | Whether relation can be null                                                                |
| innerKey    | string\|array | null      | Key column(s) in source entity (parent). Defaults to source's primary key                   |
| outerKey    | string\|array | null      | Foreign key column(s) in target entity. Defaults to `{relationName}_{innerKey}`             |
| fkCreate    | bool          | true      | Automatically create foreign key constraint                                                 |
| fkAction    | string        | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'           |
| fkOnDelete  | string        | null      | Foreign key DELETE action (overrides `fkAction` if set): 'CASCADE', 'NO ACTION', 'SET NULL' |
| indexCreate | bool          | true      | Automatically create index on foreign key column(s)                                         |
| inverse     | Inverse       | null      | Configure inverse relation on target entity. See [Inverse Relations](#inverse-relations)    |

> Since Cycle ORM v2.x, `innerKey` and `outerKey` can be arrays for [composite keys](/docs/en/advanced/composite-pk.md).

### Key Behavior

RefersTo stores the foreign key in the **target** entity (not the source), using the pattern
`{relationPropertyName}_{sourcePrimaryKey}`:

```php
#[Entity]
class User
{
    // Creates column in Comment: last_comment_id (references comment.id)
    #[RefersTo(target: Comment::class)]
    private ?Comment $lastComment;
}
```

This is different from BelongsTo, which stores the FK in the source entity.

## Use Cases

### Multiple Relations to Same Entity

RefersTo is ideal when you need multiple relations pointing to the same entity:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\RefersTo;

#[Entity]
class Document
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;

    #[RefersTo(target: User::class)]
    private User $createdBy;

    #[RefersTo(target: User::class)]
    private ?User $modifiedBy = null;

    #[RefersTo(target: User::class)]
    private ?User $approvedBy = null;

    public function __construct(string $title, User $createdBy)
    {
        $this->title = $title;
        $this->createdBy = $createdBy;
    }

    public function modify(User $user): void
    {
        $this->modifiedBy = $user;
    }

    public function approve(User $user): void
    {
        $this->approvedBy = $user;
    }
}
```

This creates three separate FK columns in the target User table:

- `created_by_id`
- `modified_by_id`
- `approved_by_id`

### Self-Referencing Relations

RefersTo is perfect for hierarchical or tree structures:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\RefersTo;

#[Entity]
class Category
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    #[RefersTo(target: Category::class, nullable: true)]
    private ?Category $parent = null;

    public function __construct(string $name, ?Category $parent = null)
    {
        $this->name = $name;
        $this->parent = $parent;
    }

    public function getParent(): ?Category
    {
        return $this->parent;
    }

    public function setParent(?Category $parent): void
    {
        $this->parent = $parent;
    }
}
```

Build hierarchies:

```php
$electronics = new Category("Electronics");
$computers = new Category("Computers", $electronics);
$laptops = new Category("Laptops", $computers);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($electronics);
$manager->persist($computers);
$manager->persist($laptops);
$manager->run();
```

## Usage Examples

### Creating Relations

The ORM will automatically save the related entity and link to it (unless `cascade: false`):

```php
$user = new User();
$user->setUsername("johndoe");

$comment = new Comment("Great article!");
$user->addComment($comment);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

**Persist Order:**

1. Source entity (`User`) is persisted first (without FK reference)
2. Target entity (`Comment`) is persisted with generated ID
3. Source entity is updated with FK reference to target

### Updating Relations

Change the relation reference:

```php
$user = $orm->getRepository(User::class)->findByPK(1);

$newComment = new Comment("Updated comment");
$user->addComment($newComment);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

### Removing Relations

Set the reference to `null`:

```php
$user = $orm->getRepository(User::class)->findByPK(1);
$user->removeLastComment();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

## Loading

### Explicit Loading

Load the related entity explicitly:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('lastComment')
    ->wherePK(1)
    ->fetchOne();

print_r($user->getLastComment());
```

### Deep Loading

For self-referencing relations, load multiple levels:

```php
$category = $orm->getRepository(Category::class)
    ->select()
    ->load('parent.parent.parent.parent') // Load 4 parent levels
    ->wherePK(1)
    ->fetchOne();

// Navigate hierarchy
$current = $category;
while ($current->getParent() !== null) {
    echo $current->getParent()->getName() . " > ";
    $current = $current->getParent();
}
```

### Eager Loading

Configure automatic loading:

```php
#[RefersTo(target: Comment::class, load: 'eager')]
private ?Comment $lastComment;
```

```php
$user = $orm->getRepository(User::class)->findByPK(1);
// lastComment is already loaded
print_r($user->getLastComment());
```

## Filtering

### Using `with()` for Filtering

Filter source entities based on target entity criteria:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('lastComment')->where('lastComment.approved', true)
    ->fetchAll();
```

### Automatic Joins

The Select query automatically joins when you reference the relation:

```php
// Automatically joins the comments table
$users = $orm->getRepository(User::class)
    ->select()
    ->where('lastComment.approved', true)
    ->where('lastComment.rating', '>', 4)
    ->fetchAll();
```

### Self-Referencing Filtering

Find categories by parent criteria:

```php
$categories = $orm->getRepository(Category::class)
    ->select()
    ->where('parent.name', 'Electronics')
    ->fetchAll();
```

Find top-level categories (no parent):

```php
$topLevelCategories = $orm->getRepository(Category::class)
    ->select()
    ->where('parent.id', null)
    ->fetchAll();
```

## Inverse Relations

Define bidirectional relationships using the `inverse` parameter:

```php
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class User
{
    #[RefersTo(
        target: Comment::class,
        inverse: new Inverse(as: 'user', type: 'belongsTo')
    )]
    private ?Comment $lastComment;
}
```

This creates the inverse relation on Comment:

```php
// Equivalent to defining on Comment:
#[Entity]
class Comment
{
    #[BelongsTo(target: User::class)]
    private User $user;
}
```

## Foreign Key Options

### Default Behavior

By default, RefersTo creates a foreign key with CASCADE:

```php
#[RefersTo(target: Comment::class)]
private ?Comment $lastComment;
// Creates: FOREIGN KEY (last_comment_id) REFERENCES comments(id) 
//          ON DELETE CASCADE ON UPDATE CASCADE
```

### Custom Foreign Key Actions

```php
#[RefersTo(
    target: Comment::class,
    fkAction: 'SET NULL',
    nullable: true
)]
private ?Comment $lastComment;
```

### Separate DELETE Action

```php
#[RefersTo(
    target: Comment::class,
    fkAction: 'CASCADE',
    fkOnDelete: 'SET NULL',
    nullable: true
)]
private ?Comment $lastComment;
```

**Available Actions:**

| Action    | Description                                                         |
|-----------|---------------------------------------------------------------------|
| CASCADE   | Delete/update source when target is deleted/updated                 |
| SET NULL  | Set foreign key to NULL (requires `nullable: true`)                 |
| NO ACTION | Prevent target deletion/update if source exists (database enforced) |

### Disable Foreign Key Creation

```php
#[RefersTo(
    target: Comment::class,
    fkCreate: false,
    indexCreate: false
)]
private ?Comment $lastComment;
```

## Best Practices

1. **Use RefersTo for multiple relations** to the same entity instead of BelongsTo
2. **Always set nullable: true** for self-referencing relations to allow root nodes
3. **Be cautious with cascade deletion** on self-referencing structures
4. **Consider the persist order** - RefersTo requires an extra update query
5. **Use explicit loading** to avoid N+1 query problems with deep hierarchies
6. **Validate circular references** in self-referencing structures to prevent infinite loops

## Common Patterns

### Tree Navigation Helper

```php
#[Entity]
class Category
{
    #[RefersTo(target: Category::class, nullable: true)]
    private ?Category $parent = null;

    public function getAncestors(): array
    {
        $ancestors = [];
        $current = $this->parent;
        
        while ($current !== null) {
            $ancestors[] = $current;
            $current = $current->getParent();
        }
        
        return $ancestors;
    }

    public function getPath(): array
    {
        return array_reverse($this->getAncestors());
    }

    public function isChildOf(Category $category): bool
    {
        return in_array($category, $this->getAncestors(), true);
    }
}
```

### Audit Trail Pattern

```php
#[Entity]
class Document
{
    #[RefersTo(target: User::class)]
    private User $createdBy;

    #[RefersTo(target: User::class, nullable: true)]
    private ?User $modifiedBy = null;

    #[Column(type: 'datetime')]
    private \DateTimeInterface $createdAt;

    #[Column(type: 'datetime', nullable: true)]
    private ?\DateTimeInterface $modifiedAt = null;

    public function __construct(User $creator)
    {
        $this->createdBy = $creator;
        $this->createdAt = new \DateTimeImmutable();
    }

    public function modify(User $user): void
    {
        $this->modifiedBy = $user;
        $this->modifiedAt = new \DateTimeImmutable();
    }
}
```

### Preventing Circular References

```php
#[Entity]
class Category
{
    #[RefersTo(target: Category::class, nullable: true)]
    private ?Category $parent = null;

    public function setParent(?Category $parent): void
    {
        if ($parent !== null && $parent->isChildOf($this)) {
            throw new \InvalidArgumentException(
                'Cannot set parent: would create circular reference'
            );
        }
        
        $this->parent = $parent;
    }

    private function isChildOf(Category $category): bool
    {
        $current = $this->parent;
        
        while ($current !== null) {
            if ($current === $category) {
                return true;
            }
            $current = $current->getParent();
        }
        
        return false;
    }
}
```

## Differences from BelongsTo

| Aspect           | RefersTo                             | BelongsTo                        |
|------------------|--------------------------------------|----------------------------------|
| Foreign Key      | Stored in target entity              | Stored in source entity          |
| Save Order       | Source → Target → Update Source      | Parent → Child                   |
| Primary Use Case | Multiple refs to same entity, cycles | Single parent-child relationship |
| Update Queries   | Requires extra update                | Single insert                    |
| Self-Reference   | Fully supported                      | Not recommended                  |

## Performance Considerations

### Extra Update Query

RefersTo requires an additional UPDATE query:

```sql
-- 1. Insert source
INSERT INTO users (username)
VALUES ('john');

-- 2. Insert target
INSERT INTO comments (text)
VALUES ('Great!');

-- 3. Update source with reference
UPDATE users
SET last_comment_id = 1
WHERE id = 1;
```

For high-volume scenarios, consider if BelongsTo would work instead.

### Deep Hierarchies

Loading deep self-referencing structures can cause performance issues:

```php
// Bad: Loads one level at a time
$category = $orm->getRepository(Category::class)->findByPK(1);
while ($category->getParent() !== null) {
    $category = $category->getParent(); // N queries!
}

// Good: Load all at once
$category = $orm->getRepository(Category::class)
    ->select()
    ->load('parent.parent.parent') // Single query
    ->wherePK(1)
    ->fetchOne();
```

## See Also

- [BelongsTo Relations](/docs/en/relation/belongs-to.md) - Similar relation with different FK placement
- [HasOne Relations](/docs/en/relation/has-one.md) - One-to-one relationships
- [HasMany Relations](/docs/en/relation/has-many.md) - One-to-many relationships
- [Morphed Relations](/docs/en/relation/morphed.md) - RefersToMorphed for polymorphic self/cyclic references
- [Composite Keys](/docs/en/advanced/composite-pk.md) - Using composite keys
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced querying
