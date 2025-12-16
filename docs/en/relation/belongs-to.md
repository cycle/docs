# Belongs To

**Belongs To** relation defines that an entity is owned by a related entity on an exclusive basis. **Example:** a post
belongs to an author, a comment belongs to a post. Most `belongsTo` relations can be created using the `inverse` option of
a declared `hasOne` or `hasMany` relation.

> The child entity will always be persisted **after** its parent entity to ensure referential integrity.

## Table of Contents

- [Definition](#definition)
  - [Attribute Specification](#attribute-specification)
  - [Key Behavior](#key-behavior)
- [Usage Examples](#usage-examples)
  - [Creating Relations](#creating-relations)
  - [Nullable Relations](#nullable-relations)
  - [Composite Keys](#composite-keys)
- [Loading](#loading)
- [Filtering](#filtering)
- [Inverse Relations](#inverse-relations)
- [Foreign Key Options](#foreign-key-options)

## Definition

To define a BelongsTo relation using the annotated entities extension:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\BelongsTo;

#[Entity]
class Post
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;

    #[BelongsTo(target: User::class)]
    private User $author;

    public function __construct(string $title, User $author)
    {
        $this->title = $title;
        $this->author = $author;
    }

    public function getAuthor(): User
    {
        return $this->author;
    }

    public function setAuthor(User $author): void
    {
        $this->author = $author;
    }
}
```

### Attribute Specification

| Parameter   | Type          | Default                           | Description                                                                                                    |
|-------------|---------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------|
| target      | string        | -                                 | **Required**. Target entity role or class name                                                                 |
| load        | string        | 'lazy'                            | Loading strategy: 'lazy' or 'eager'                                                                            |
| cascade     | bool          | true                              | Automatically save parent entity with child entity                                                             |
| nullable    | bool          | false                             | Whether relation can be null (child can exist without parent)                                                  |
| innerKey    | string\|array | null                              | Foreign key column(s) in child entity. Defaults to `{relationName}_{outerKey}`                                 |
| outerKey    | string\|array | null                              | Key column(s) in parent entity. Defaults to parent's primary key                                               |
| fkCreate    | bool          | true                              | Automatically create foreign key constraint                                                                    |
| fkAction    | string        | 'CASCADE'                         | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'                              |
| fkOnDelete  | string        | null                              | Foreign key DELETE action (overrides `fkAction` if set): 'CASCADE', 'NO ACTION', 'SET NULL'                    |
| indexCreate | bool          | true                              | Automatically create index on foreign key column(s)                                                            |
| inverse     | Inverse       | null                              | Configure inverse relation on parent entity. See [Inverse Relations](#inverse-relations)                       |

> Since Cycle ORM v2.x, `innerKey` and `outerKey` can be arrays for [composite keys](/docs/en/advanced/composite-pk.md).

### Key Behavior

By default, the ORM will generate a foreign key column in the child entity using the pattern `{relationPropertyName}_{parentPrimaryKey}`:

```php
#[Entity]
class Post
{
    // Creates column: author_id (references user.id)
    #[BelongsTo(target: User::class)]
    private User $author;
}
```

You can customize column names explicitly:

```php
#[Entity]
class Post
{
    // Uses existing column: user_id
    #[BelongsTo(target: User::class, innerKey: 'user_id')]
    private User $author;
}
```

## Usage Examples

### Creating Relations

The ORM will automatically save the parent entity when persisting the child (unless `cascade` is set to `false`):

```php
$user = new User("John Doe");
$post = new Post("My First Post", $user);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->run();
```

The save order ensures that:
1. The parent (`User`) is saved first, generating its ID
2. The child (`Post`) is saved second, with the parent's ID as the foreign key

### Nullable Relations

You can only set the relation to `null` if it's defined as `nullable`:

```php
#[Entity]
class Post
{
    #[BelongsTo(target: User::class, nullable: true)]
    private ?User $author = null;

    public function removeAuthor(): void
    {
        $this->author = null;
    }
}
```

```php
$post = $orm->getRepository(Post::class)->findByPK(1);
$post->removeAuthor();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->run(); // Sets author_id to NULL
```

Without `nullable: true`, attempting to set the relation to `null` will cause a database integrity constraint violation.

### Composite Keys

BelongsTo supports composite keys for complex relationships:

```php
#[Entity]
class OrderItem
{
    #[BelongsTo(
        target: Order::class,
        innerKey: ['order_id', 'order_date'],
        outerKey: ['id', 'created_date']
    )]
    private Order $order;
}
```

> Read more about [composite keys](/docs/en/advanced/composite-pk.md).

## Loading

### Explicit Loading

To load related data, use the `load` method on your repository's Select query:

```php
$post = $orm->getRepository(Post::class)
    ->select()
    ->load('author')
    ->wherePK(1)
    ->fetchOne();

print_r($post->getAuthor());
```

### Eager Loading

Configure the relation to always load with the entity:

```php
#[BelongsTo(target: User::class, load: 'eager')]
private User $author;
```

With eager loading, the parent is automatically loaded without calling `load()`:

```php
$post = $orm->getRepository(Post::class)->findByPK(1);
// Author is already loaded
print_r($post->getAuthor());
```

### Loading Multiple Relations

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->load('author')
    ->load('category')
    ->load('comments')
    ->fetchAll();
```

## Filtering

### Using `with()` for Filtering

Filter the parent entity selection based on related entity criteria using `with()`:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->with('author')->where('author.status', 'active')
    ->fetchAll();
```

### Automatic Joins

The Select query automatically joins related tables when you reference them in `where()` conditions:

```php
// Automatically joins the user table
$posts = $orm->getRepository(Post::class)
    ->select()
    ->where('author.status', 'active')
    ->where('author.email', 'like', '%@example.com')
    ->fetchAll();
```

### Complex Filtering

Combine multiple conditions and nested relations:

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->where('author.status', 'active')
    ->where('author.role', 'editor')
    ->where('published_at', '>', new DateTime('-30 days'))
    ->orderBy('author.name')
    ->fetchAll();
```

## Inverse Relations

Define bidirectional relationships using the `inverse` parameter:

```php
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class User
{
    #[HasMany(
        target: Post::class,
        inverse: new Inverse(as: 'author', type: 'belongsTo')
    )]
    private array $posts = [];
}
```

This automatically creates the inverse BelongsTo relation on Post:

```php
// Equivalent to defining on Post:
#[Entity]
class Post
{
    #[BelongsTo(target: User::class)]
    private User $author;
}
```

**Inverse Attribute Parameters:**

| Parameter | Type   | Default | Description                                                      |
|-----------|--------|---------|------------------------------------------------------------------|
| as        | string | -       | **Required**. Property name for the inverse relation             |
| type      | string | -       | **Required**. Relation type: 'belongsTo', 'hasOne', 'hasMany', etc. |
| load      | string | null    | Loading strategy for inverse: 'eager', 'lazy', 'promise'         |

## Foreign Key Options

### Default Behavior

By default, BelongsTo creates a foreign key with CASCADE on both DELETE and UPDATE:

```php
#[BelongsTo(target: User::class)]
private User $author;
// Creates: FOREIGN KEY (author_id) REFERENCES users(id) 
//          ON DELETE CASCADE ON UPDATE CASCADE
```

### Custom Foreign Key Actions

Control what happens when the parent is deleted or updated:

```php
#[BelongsTo(
    target: User::class,
    fkAction: 'SET NULL',      // Both DELETE and UPDATE
    nullable: true              // Required for SET NULL
)]
private ?User $author;
```

### Separate DELETE Action

Set different behavior for DELETE operations:

```php
#[BelongsTo(
    target: User::class,
    fkAction: 'CASCADE',        // Default for UPDATE
    fkOnDelete: 'SET NULL',     // Custom for DELETE
    nullable: true              // Required for SET NULL
)]
private ?User $author;
```

**Available Actions:**

| Action    | Description                                                          |
|-----------|----------------------------------------------------------------------|
| CASCADE   | Delete/update child when parent is deleted/updated                   |
| SET NULL  | Set foreign key to NULL (requires `nullable: true`)                  |
| NO ACTION | Prevent parent deletion/update if children exist (database enforced) |

### Disable Foreign Key Creation

For scenarios where you need manual control or cross-database relations:

```php
#[BelongsTo(
    target: User::class,
    fkCreate: false,
    indexCreate: false  // Often combined with fkCreate: false
)]
private User $author;
```

### Manual Column Definition

Explicitly define the foreign key column:

```php
#[Entity]
class Post
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'integer', nullable: false)]
    private int $author_id;

    #[BelongsTo(target: User::class, innerKey: 'author_id')]
    private User $author;
}
```

## Best Practices

1. **Always initialize required relations** in the constructor or factory methods
2. **Use nullable relations sparingly** - prefer required relationships when possible
3. **Consider cascade behavior** carefully based on your business logic
4. **Use explicit loading** (`load()`) instead of eager loading for better performance control
5. **Leverage automatic joins** in queries - they're more efficient than loading relations separately

## Common Patterns

### Optional Parent with Default

```php
#[Entity]
class Post
{
    #[BelongsTo(target: User::class, nullable: true)]
    private ?User $author = null;

    public function getAuthor(): User
    {
        return $this->author ?? $this->getDefaultAuthor();
    }

    private function getDefaultAuthor(): User
    {
        // Return system user or guest user
    }
}
```

### Change Tracking

```php
#[Entity]
class Post
{
    #[BelongsTo(target: User::class)]
    private User $author;

    public function changeAuthor(User $newAuthor): void
    {
        if ($this->author->getId() !== $newAuthor->getId()) {
            // Log the change, notify, etc.
            $this->author = $newAuthor;
        }
    }
}
```

## See Also

- [HasOne Relations](/docs/en/relation/has-one.md) - The inverse of BelongsTo
- [HasMany Relations](/docs/en/relation/has-many.md) - One-to-many relationships
- [RefersTo Relations](/docs/en/relation/refers-to.md) - For multiple relations to the same entity
- [Composite Keys](/docs/en/advanced/composite-pk.md) - Using composite keys in relations
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced relation querying
