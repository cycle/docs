# Has Many

The **Has Many** relation defines that an entity exclusively owns multiple other entities in a parent-children
relationship. This is the most common one-to-many relationship pattern.

> The parent entity is persisted first, then all child entities with references to the parent.

## Table of Contents

- [Definition](#definition)
    - [Attribute Specification](#attribute-specification)
    - [Key Behavior](#key-behavior)
- [Usage Examples](#usage-examples)
    - [Creating Relations](#creating-relations)
    - [Managing Collections](#managing-collections)
    - [Removing Children](#removing-children)
- [Loading](#loading)
    - [Filtered Loading](#filtered-loading)
    - [Sorted Loading](#sorted-loading)
- [Filtering](#filtering)
- [Collections](#collections)
- [Inverse Relations](#inverse-relations)
- [Foreign Key Options](#foreign-key-options)

## Definition

To define a HasMany relation using the annotated entities extension:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $username;

    #[HasMany(target: Post::class)]
    private array $posts = [];

    public function getPosts(): array
    {
        return $this->posts;
    }

    public function addPost(Post $post): void
    {
        $this->posts[] = $post;
    }

    public function removePost(Post $post): void
    {
        $this->posts = array_filter(
            $this->posts, 
            static fn(Post $p) => $p !== $post
        );
    }
}

#[Entity]
class Post
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;
}
```

### Attribute Specification

| Parameter   | Type          | Default   | Description                                                                                 |
|-------------|---------------|-----------|---------------------------------------------------------------------------------------------|
| target      | string        | -         | **Required**. Target entity role or class name                                              |
| load        | string        | 'lazy'    | Loading strategy: 'lazy' or 'eager'                                                         |
| cascade     | bool          | true      | Automatically save child entities with parent entity                                        |
| nullable    | bool          | false     | Whether children can exist without parent (affects foreign key NULL constraint)             |
| innerKey    | string\|array | null      | Key column(s) in parent entity. Defaults to parent's primary key                            |
| outerKey    | string\|array | null      | Foreign key column(s) in child entities. Defaults to `{parentRole}_{innerKey}`              |
| where       | array         | []        | Additional WHERE conditions applied when loading relation                                   |
| orderBy     | array         | []        | Default sorting for loaded collection                                                       |
| fkCreate    | bool          | true      | Automatically create foreign key constraint                                                 |
| fkAction    | string        | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'           |
| fkOnDelete  | string        | null      | Foreign key DELETE action (overrides `fkAction` if set): 'CASCADE', 'NO ACTION', 'SET NULL' |
| indexCreate | bool          | true      | Automatically create index on foreign key column(s)                                         |
| collection  | string        | null      | Collection class for loaded entities. See [Collections](#collections)                       |
| inverse     | Inverse       | null      | Configure inverse relation on child entities. See [Inverse Relations](#inverse-relations)   |

> Since Cycle ORM v2.x, `innerKey` and `outerKey` can be arrays for [composite keys](/docs/en/advanced/composite-pk.md).

### Key Behavior

By default, the ORM generates a foreign key column in child entities using the pattern
`{parentRole}_{parentPrimaryKey}`:

```php
#[Entity]
class User
{
    // Creates column in Post: user_id (references user.id)
    #[HasMany(target: Post::class)]
    private array $posts = [];
}
```

Customize the foreign key column:

```php
#[Entity]
class User
{
    // Uses custom column in Post: author_id
    #[HasMany(target: Post::class, outerKey: 'author_id')]
    private array $posts = [];
}
```

## Usage Examples

### Creating Relations

Child entities are automatically saved with the parent when persisted (unless `cascade: false`):

```php
$user = new User();
$user->setUsername("johndoe");

$post1 = new Post();
$post1->setTitle("First Post");

$post2 = new Post();
$post2->setTitle("Second Post");

$user->addPost($post1);
$user->addPost($post2);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

The save order ensures:

1. Parent (`User`) is saved first, generating its ID
2. All children (`Post` entities) are saved, with parent's ID as the foreign key

### Managing Collections

#### Adding Children

```php
$user = $orm->getRepository(User::class)->findByPK(1);

$newPost = new Post();
$newPost->setTitle("Another Post");

$user->addPost($newPost);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

#### Replacing Collection

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('posts')
    ->wherePK(1)
    ->fetchOne();

// Create new collection
$newPosts = [
    new Post("Post 1"),
    new Post("Post 2"),
];

// This will delete old posts and create new ones
$user->setPosts($newPosts);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

### Removing Children

To delete a child entity, remove it from the collection:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('posts')
    ->wherePK(1)
    ->fetchOne();

$postToRemove = $user->getPosts()[0];
$user->removePost($postToRemove);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run(); // Post is deleted
```

#### Detaching Instead of Deleting

If you want to detach children without deleting them, use `nullable: true`:

```php
#[HasMany(target: Post::class, nullable: true)]
private array $posts = [];
```

```php
$user->removePost($post);
// Post is detached (user_id set to NULL) but not deleted
```

## Loading

### Basic Loading

Load child entities explicitly using the `load()` method:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('posts')
    ->wherePK(1)
    ->fetchOne();

foreach ($user->getPosts() as $post) {
    echo $post->getTitle() . "\n";
}
```

### Eager Loading

Configure the relation to always load with the parent:

```php
#[HasMany(target: Post::class, load: 'eager')]
private array $posts = [];
```

With eager loading:

```php
$user = $orm->getRepository(User::class)->findByPK(1);
// Posts are already loaded
print_r($user->getPosts());
```

> **Note:** By default, HasMany loads children using a separate `WHERE IN` query, not a JOIN. This is more efficient for
> one-to-many relationships.

### Filtered Loading

Pre-filter child entities when loading:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('posts', ['where' => ['published' => true]])
    ->fetchAll();
```

You can also set default filters in the relation definition:

```php
#[HasMany(
    target: Post::class,
    where: ['published' => true, 'deleted_at' => null]
)]
private array $posts = [];
```

### Sorted Loading

Specify sorting when loading:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('posts', [
        'where' => ['published' => true],
        'orderBy' => ['published_at' => 'DESC']
    ])
    ->fetchAll();
```

Set default sorting in the relation definition:

```php
#[HasMany(
    target: Post::class,
    orderBy: ['published_at' => 'DESC', 'title' => 'ASC']
)]
private array $posts = [];
```

### Using `with` for Preloading

Combine `with()` and `load()` for single-query loading:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts', ['as' => 'published_posts'])
        ->where('posts.published', true)
    ->load('posts', ['using' => 'published_posts'])
    ->fetchAll();
```

This produces a single SQL query instead of separate queries.

## Filtering

### Using `with()` for Filtering

Filter parent entities based on child entity criteria:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')->where('posts.published', true)
    ->fetchAll();
```

> **Important:** Always use `distinct()` with HasMany filtering to avoid duplicate parent rows.

### Automatic Joins

The Select query automatically joins related tables when referenced:

```php
// Automatically joins the posts table
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('posts.published', true)
    ->where('posts.views', '>', 1000)
    ->fetchAll();
```

### Complex Filtering

Find users with at least 5 published posts:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')
    ->where('posts.published', true)
    ->having('COUNT(posts.id)', '>=', 5)
    ->fetchAll();
```

### Filtering on NULL Relations

Find users without posts:

```php
$usersWithoutPosts = $orm->getRepository(User::class)
    ->select()
    ->where('posts.id', null)
    ->fetchAll();
```

## Collections

HasMany relations support custom collection classes for managing child entities. By default, relations use a simple
array, but you can use more sophisticated collection types.

### Default Array Collection

```php
#[HasMany(target: Post::class)]
private array $posts = [];
```

### Doctrine Collection

Use Doctrine Collections for richer functionality:

```php
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;

#[Entity]
class User
{
    #[HasMany(target: Post::class, collection: 'doctrine')]
    private Collection $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    public function getPosts(): Collection
    {
        return $this->posts;
    }

    public function addPost(Post $post): void
    {
        if (!$this->posts->contains($post)) {
            $this->posts->add($post);
        }
    }
}
```

### Custom Collection Factory

```php
#[HasMany(target: Post::class, collection: 'my_custom_collection')]
private CustomCollection $posts;
```

> Read more about [relation collections](/docs/en/relation/collections.md).

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

| Parameter | Type   | Default | Description                                                         |
|-----------|--------|---------|---------------------------------------------------------------------|
| as        | string | -       | **Required**. Property name for the inverse relation                |
| type      | string | -       | **Required**. Relation type: 'belongsTo', 'hasOne', 'hasMany', etc. |
| load      | string | null    | Loading strategy for inverse: 'eager', 'lazy', 'promise'            |

## Foreign Key Options

### Default Behavior

By default, HasMany creates a foreign key with CASCADE on both DELETE and UPDATE:

```php
#[HasMany(target: Post::class)]
private array $posts = [];
// Creates: FOREIGN KEY (user_id) REFERENCES users(id) 
//          ON DELETE CASCADE ON UPDATE CASCADE
```

When a user is deleted, all their posts are automatically deleted.

### Custom Foreign Key Actions

Control what happens when the parent is deleted or updated:

```php
#[HasMany(
    target: Post::class,
    fkAction: 'SET NULL',      // Both DELETE and UPDATE
    nullable: true              // Required for SET NULL
)]
private array $posts = [];
```

### Separate DELETE Action

```php
#[HasMany(
    target: Post::class,
    fkAction: 'CASCADE',        // Default for UPDATE
    fkOnDelete: 'NO ACTION',    // Prevent deletion if posts exist
)]
private array $posts = [];
```

**Available Actions:**

| Action    | Description                                                          |
|-----------|----------------------------------------------------------------------|
| CASCADE   | Delete/update children when parent is deleted/updated                |
| SET NULL  | Set foreign key to NULL (requires `nullable: true`)                  |
| NO ACTION | Prevent parent deletion/update if children exist (database enforced) |

### Disable Foreign Key Creation

```php
#[HasMany(
    target: Post::class,
    fkCreate: false,
    indexCreate: false
)]
private array $posts = [];
```

## Best Practices

1. **Always initialize collections** to empty arrays in property declarations
2. **Use distinct()** when filtering by child properties to avoid duplicate parents
3. **Consider cascade deletion carefully** - ensure you want children deleted with parent
4. **Use explicit loading** instead of eager loading for better performance control
5. **Implement proper add/remove methods** to maintain collection consistency
6. **Use WHERE and ORDER BY** in relation definitions for default filtering/sorting

## Common Patterns

### Conditional Loading

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('posts', [
        'where' => [
            'published' => true,
            'published_at' => ['>' => new DateTime('-30 days')]
        ]
    ])
    ->fetchAll();
```

### Counting Children

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('posts')
    ->having('COUNT(posts.id)', '>', 10)
    ->fetchAll();
```

### Polymorphic Collections

Use different collection strategies:

```php
#[Entity]
class User
{
    #[HasMany(
        target: Post::class,
        where: ['published' => true],
        orderBy: ['published_at' => 'DESC']
    )]
    private array $publishedPosts = [];

    #[HasMany(
        target: Post::class,
        where: ['published' => false]
    )]
    private array $draftPosts = [];
}
```

### Batch Operations

```php
$user = $orm->getRepository(User::class)->findByPK(1);

// Add multiple posts efficiently
$newPosts = [
    new Post("Post 1"),
    new Post("Post 2"),
    new Post("Post 3"),
];

foreach ($newPosts as $post) {
    $user->addPost($post);
}

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run(); // Single transaction
```

## Performance Considerations

### Loading Strategy

```php
// Separate query (default) - better for many children
$user = $orm->getRepository(User::class)
    ->select()
    ->load('posts')
    ->wherePK(1)
    ->fetchOne();

// Single query with JOIN - better for few children or filtering
$user = $orm->getRepository(User::class)
    ->select()
    ->with('posts')
    ->load('posts', ['using' => 'posts'])
    ->wherePK(1)
    ->fetchOne();
```

### Pagination

When dealing with large collections, use pagination on the child query:

```php
$user = $orm->getRepository(User::class)->findByPK(1);

$posts = $orm->getRepository(Post::class)
    ->select()
    ->where('user_id', $user->getId())
    ->orderBy('created_at', 'DESC')
    ->limit(20)
    ->offset(0)
    ->fetchAll();
```

## See Also

- [BelongsTo Relations](/docs/en/relation/belongs-to.md) - The inverse of HasMany
- [HasOne Relations](/docs/en/relation/has-one.md) - One-to-one relationships
- [ManyToMany Relations](/docs/en/relation/many-to-many.md) - Many-to-many relationships
- [Relation Collections](/docs/en/relation/collections.md) - Collection management
- [Composite Keys](/docs/en/advanced/composite-pk.md) - Using composite keys
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced querying
