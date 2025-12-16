# Morphed Relations

**Morphed (polymorphic) relations** allow an entity to belong to or own entities of multiple types through a single
relation. These relations use interfaces to define the contract that multiple entity types can fulfill.

> **Use with caution:** Polymorphic associations can make queries more complex and prevent proper foreign key
> constraints. Consider alternative designs when possible.

## Table of Contents

- [Overview](#overview)
- [Definition](#definition)
- [BelongsToMorphed](#belongstomorphed)
    - [Attribute Specification](#belongstomorphed-attribute-specification)
    - [Usage Examples](#belongstomorphed-usage-examples)
- [MorphedHasOne](#morphedhasone)
    - [Attribute Specification](#morphedhasone-attribute-specification)
    - [Usage Examples](#morphedhasone-usage-examples)
- [MorphedHasMany](#morphedhasmany)
    - [Attribute Specification](#morphedhasmany-attribute-specification)
    - [Usage Examples](#morphedhasmany-usage-examples)
- [Loading](#loading)
- [Filtering](#filtering)
- [Collections](#collections)
- [Inverse Relations](#inverse-relations)
- [Best Practices](#best-practices)
- [Limitations](#limitations)

## Overview

Morphed relations solve the problem of entities that can relate to multiple entity types:

- **Example 1:** Comments that can be attached to Posts, Videos, or Articles
- **Example 2:** Images that belong to Users, Products, or Categories
- **Example 3:** Notifications for various entity types

Instead of creating separate relation properties for each type, morphed relations use:

- A **role column** (morph key) to store the entity type
- A **foreign key column** to store the entity ID
- An **interface** as the target

## Definition

First, define a common interface for all entities that can participate:

```php
interface CommentableInterface
{
    public function getId(): int;
}
```

Then implement it on your entities:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class Post implements CommentableInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;

    public function getId(): int
    {
        return $this->id;
    }
}

#[Entity]
class Video implements CommentableInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $url;

    public function getId(): int
    {
        return $this->id;
    }
}
```

## BelongsToMorphed

**BelongsToMorphed** allows an entity to belong to any parent type that implements the target interface. The child
entity stores the morph key and foreign key.

### BelongsToMorphed Attribute Specification

| Parameter      | Type          | Default | Description                                                                                |
|----------------|---------------|---------|--------------------------------------------------------------------------------------------|
| target         | string        | -       | **Required**. Target interface name                                                        |
| load           | string        | 'lazy'  | Loading strategy: 'lazy' or 'eager'                                                        |
| cascade        | bool          | true    | Automatically save parent entity with child entity                                         |
| nullable       | bool          | true    | Whether relation can be null                                                               |
| innerKey       | string\|array | null    | Foreign key column(s) in child entity. Defaults to `{relationName}_{outerKey}`             |
| outerKey       | string\|array | null    | Key column(s) in parent entity. Defaults to parent's primary key                           |
| morphKey       | string        | null    | Column storing parent entity type (role). Defaults to `{relationName}_role`                |
| morphKeyLength | int           | 32      | Length of morph key column                                                                 |
| indexCreate    | bool          | true    | Create index on [morphKey, innerKey] for query performance                                 |
| inverse        | Inverse       | null    | Configure inverse relation on parent entities. See [Inverse Relations](#inverse-relations) |

> **Note:** Foreign key constraints are not supported for morphed relations since the target can be multiple entity
> types.

### BelongsToMorphed Usage Examples

#### Definition

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed;

#[Entity]
class Comment
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'text')]
    private string $content;

    #[BelongsToMorphed(target: CommentableInterface::class)]
    private CommentableInterface $commentable;

    public function __construct(string $content, CommentableInterface $commentable)
    {
        $this->content = $content;
        $this->commentable = $commentable;
    }

    public function getCommentable(): CommentableInterface
    {
        return $this->commentable;
    }

    public function setCommentable(CommentableInterface $commentable): void
    {
        $this->commentable = $commentable;
    }
}
```

This creates in the `comments` table:

- `commentable_id` - stores the parent entity ID
- `commentable_role` - stores the parent entity role ("post", "video", etc.)

#### Creating Relations

```php
$post = new Post("Introduction to Cycle ORM");
$comment1 = new Comment("Great article!", $post);

$video = new Video("https://example.com/tutorial.mp4");
$comment2 = new Comment("Very helpful!", $video);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->persist($comment1);
$manager->persist($video);
$manager->persist($comment2);
$manager->run();
```

#### Accessing Relations

```php
$comment = $orm->getRepository(Comment::class)
    ->select()
    ->load('commentable')
    ->wherePK(1)
    ->fetchOne();

$parent = $comment->getCommentable();

if ($parent instanceof Post) {
    echo "Comment on post: " . $parent->getTitle();
} elseif ($parent instanceof Video) {
    echo "Comment on video: " . $parent->getUrl();
}
```

#### Nullable Relations

```php
#[BelongsToMorphed(target: CommentableInterface::class, nullable: true)]
private ?CommentableInterface $commentable = null;
```

```php
$comment->setCommentable(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($comment);
$manager->run();
```

## MorphedHasOne

**MorphedHasOne** is the inverse of BelongsToMorphed for one-to-one relationships. The parent entity owns one child
through a polymorphic relation.

### MorphedHasOne Attribute Specification

| Parameter      | Type          | Default | Description                                                                             |
|----------------|---------------|---------|-----------------------------------------------------------------------------------------|
| target         | string        | -       | **Required**. Target entity role or class name                                          |
| load           | string        | 'lazy'  | Loading strategy: 'lazy' or 'eager'                                                     |
| cascade        | bool          | true    | Automatically save child entity with parent entity                                      |
| nullable       | bool          | false   | Whether child can exist without parent (affects morph key NULL constraint)              |
| innerKey       | string\|array | null    | Key column(s) in parent entity. Defaults to parent's primary key                        |
| outerKey       | string\|array | null    | Foreign key column(s) in child entity. Defaults to `{parentRole}_{innerKey}`            |
| morphKey       | string        | null    | Column in child storing parent entity type. Defaults to `{relationName}_role`           |
| morphKeyLength | int           | 32      | Length of morph key column                                                              |
| indexCreate    | bool          | true    | Create index on [morphKey, outerKey]                                                    |
| inverse        | Inverse       | null    | Configure inverse relation on child entity. See [Inverse Relations](#inverse-relations) |

### MorphedHasOne Usage Examples

#### Definition

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasOne;

#[Entity]
class User implements ImageHolderInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $username;

    #[MorphedHasOne(target: Image::class)]
    private ?Image $avatar = null;

    public function getAvatar(): ?Image
    {
        return $this->avatar;
    }

    public function setAvatar(?Image $avatar): void
    {
        $this->avatar = $avatar;
    }
}

#[Entity]
class Product implements ImageHolderInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    #[MorphedHasOne(target: Image::class)]
    private ?Image $thumbnail = null;
}

#[Entity]
class Image
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $url;

    public function __construct(string $url)
    {
        $this->url = $url;
    }
}
```

#### Creating Relations

```php
$user = new User();
$user->setUsername("johndoe");
$user->setAvatar(new Image("/avatars/john.jpg"));

$product = new Product();
$product->setName("Laptop");
$product->setThumbnail(new Image("/products/laptop.jpg"));

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->persist($product);
$manager->run();
```

#### Removing Relations

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('avatar')
    ->wherePK(1)
    ->fetchOne();

$user->setAvatar(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
// Image is deleted
```

## MorphedHasMany

**MorphedHasMany** is the inverse of BelongsToMorphed for one-to-many relationships. The parent entity owns multiple
children through a polymorphic relation.

### MorphedHasMany Attribute Specification

| Parameter      | Type          | Default | Description                                                                               |
|----------------|---------------|---------|-------------------------------------------------------------------------------------------|
| target         | string        | -       | **Required**. Target entity role or class name                                            |
| load           | string        | 'lazy'  | Loading strategy: 'lazy' or 'eager'                                                       |
| cascade        | bool          | true    | Automatically save child entities with parent entity                                      |
| nullable       | bool          | false   | Whether children can exist without parent                                                 |
| innerKey       | string\|array | null    | Key column(s) in parent entity. Defaults to parent's primary key                          |
| outerKey       | string\|array | null    | Foreign key column(s) in child entities. Defaults to `{parentRole}_{innerKey}`            |
| morphKey       | string        | null    | Column in children storing parent entity type. Defaults to `{relationName}_role`          |
| morphKeyLength | int           | 32      | Length of morph key column                                                                |
| where          | array         | []      | Additional WHERE conditions applied when loading                                          |
| indexCreate    | bool          | true    | Create index on [morphKey, outerKey]                                                      |
| collection     | string        | null    | Collection class for loaded entities. See [Collections](#collections)                     |
| inverse        | Inverse       | null    | Configure inverse relation on child entities. See [Inverse Relations](#inverse-relations) |

### MorphedHasMany Usage Examples

#### Definition

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasMany;

#[Entity]
class Post implements CommentableInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;

    #[MorphedHasMany(target: Comment::class)]
    private array $comments = [];

    public function getComments(): array
    {
        return $this->comments;
    }

    public function addComment(Comment $comment): void
    {
        $this->comments[] = $comment;
    }

    public function removeComment(Comment $comment): void
    {
        $this->comments = array_filter(
            $this->comments,
            static fn(Comment $c) => $c !== $comment
        );
    }
}

#[Entity]
class Video implements CommentableInterface
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $url;

    #[MorphedHasMany(target: Comment::class)]
    private array $comments = [];
}
```

#### Creating Relations

```php
$post = new Post();
$post->setTitle("Getting Started");
$post->addComment(new Comment("Great post!"));
$post->addComment(new Comment("Very helpful."));

$video = new Video();
$video->setUrl("https://example.com/tutorial.mp4");
$video->addComment(new Comment("Nice tutorial!"));

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post);
$manager->persist($video);
$manager->run();
```

#### Filtered Relations

```php
#[MorphedHasMany(
    target: Comment::class,
    where: ['approved' => true, 'deleted_at' => null],
    orderBy: ['created_at' => 'DESC']
)]
private array $comments = [];
```

## Loading

### Explicit Loading

```php
$post = $orm->getRepository(Post::class)
    ->select()
    ->load('comments')
    ->wherePK(1)
    ->fetchOne();

foreach ($post->getComments() as $comment) {
    echo $comment->getContent() . "\n";
}
```

### Eager Loading

```php
#[MorphedHasMany(target: Comment::class, load: 'eager')]
private array $comments = [];
```

### Loading Morphed Parents

For BelongsToMorphed, loading requires explicit specification:

```php
$comments = $orm->getRepository(Comment::class)
    ->select()
    ->load('commentable')
    ->fetchAll();
```

> **Note:** Eager loading is **not supported** for BelongsToMorphed relations due to the varying parent types.

## Filtering

### Filtering by Morph Type

Find comments only on posts:

```php
$postComments = $orm->getRepository(Comment::class)
    ->select()
    ->where('commentable_role', 'post')
    ->fetchAll();
```

### Filtering MorphedHas Relations

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->distinct()
    ->with('comments')->where('comments.approved', true)
    ->fetchAll();
```

> **Important:** Always use `distinct()` when filtering by morphed has-many relations.

### Complex Filtering

```php
$posts = $orm->getRepository(Post::class)
    ->select()
    ->distinct()
    ->where('comments.approved', true)
    ->having('COUNT(comments.id)', '>=', 5)
    ->fetchAll();
```

## Collections

MorphedHasMany supports custom collection types:

### Default Array

```php
#[MorphedHasMany(target: Comment::class)]
private array $comments = [];
```

### Doctrine Collection

```php
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;

#[Entity]
class Post
{
    #[MorphedHasMany(target: Comment::class, collection: 'doctrine')]
    private Collection $comments;

    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }
}
```

> Read more about [relation collections](/docs/en/relation/collections.md).

## Inverse Relations

Define bidirectional relationships:

### BelongsToMorphed with Inverse

```php
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class Post
{
    #[MorphedHasMany(
        target: Comment::class,
        inverse: new Inverse(as: 'commentable', type: 'belongsToMorphed')
    )]
    private array $comments = [];
}
```

This creates the inverse on Comment:

```php
// Equivalent to:
#[Entity]
class Comment
{
    #[BelongsToMorphed(target: CommentableInterface::class)]
    private CommentableInterface $commentable;
}
```

### MorphedHasOne with Inverse

```php
#[Entity]
class User
{
    #[MorphedHasOne(
        target: Image::class,
        inverse: new Inverse(as: 'owner', type: 'belongsToMorphed')
    )]
    private ?Image $avatar;
}
```

## Best Practices

1. **Use interfaces** to define contracts for polymorphic entities
2. **Consider alternatives** - polymorphic relations add complexity
3. **Document morph types** clearly in your codebase
4. **Use explicit loading** - eager loading doesn't work for BelongsToMorphed
5. **Index morph keys** for query performance (enabled by default)
6. **Validate parent types** before setting morphed relations
7. **Be cautious with cascade** - ensure you want children deleted with parents

## Common Patterns

### Type Checking Helper

```php
#[Entity]
class Comment
{
    #[BelongsToMorphed(target: CommentableInterface::class)]
    private CommentableInterface $commentable;

    public function isPostComment(): bool
    {
        return $this->commentable instanceof Post;
    }

    public function isVideoComment(): bool
    {
        return $this->commentable instanceof Video;
    }

    public function getParentType(): string
    {
        return match (true) {
            $this->commentable instanceof Post => 'post',
            $this->commentable instanceof Video => 'video',
            default => 'unknown',
        };
    }
}
```

### Filtered by Type

```php
#[Entity]
class Post
{
    #[MorphedHasMany(
        target: Comment::class,
        where: ['approved' => true],
        orderBy: ['created_at' => 'DESC']
    )]
    private array $approvedComments = [];

    #[MorphedHasMany(
        target: Comment::class,
        where: ['approved' => false]
    )]
    private array $pendingComments = [];
}
```

### Activity Stream

```php
interface ActivitySubjectInterface
{
    public function getId(): int;
    public function getActivityDescription(): string;
}

#[Entity]
class Activity
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $action; // 'created', 'updated', 'deleted'

    #[BelongsToMorphed(target: ActivitySubjectInterface::class)]
    private ActivitySubjectInterface $subject;

    #[BelongsTo(target: User::class)]
    private User $actor;

    public function getDescription(): string
    {
        return sprintf(
            "%s %s %s",
            $this->actor->getUsername(),
            $this->action,
            $this->subject->getActivityDescription()
        );
    }
}
```

## Limitations

### No Foreign Key Constraints

Morphed relations cannot use database foreign key constraints:

```php
// This is NOT possible:
FOREIGN KEY (commentable_id) REFERENCES ??? (id)
```

The database cannot enforce referential integrity across multiple tables.

### No Eager Loading for BelongsToMorphed

```php
#[BelongsToMorphed(target: CommentableInterface::class, load: 'eager')]
// This will NOT work - eager loading not supported
```

You must use explicit loading:

```php
$comments = $orm->getRepository(Comment::class)
    ->select()
    ->load('commentable')
    ->fetchAll();
```

### Query Complexity

Queries on morphed relations are more complex and can be slower:

```sql
-- Query must check both role and ID
SELECT *
FROM comments
WHERE commentable_role = 'post'
  AND commentable_id = 1
```

### Limited JOIN Support

JOINing polymorphic relations in queries is limited since the target table varies.

## Alternative Designs

Consider these alternatives to morphed relations:

### Separate Relations

```php
#[Entity]
class Comment
{
    #[BelongsTo(target: Post::class, nullable: true)]
    private ?Post $post = null;

    #[BelongsTo(target: Video::class, nullable: true)]
    private ?Video $video = null;
}
```

**Pros:** Foreign keys, simpler queries, better performance
**Cons:** More code, multiple nullable fields

### Intermediate Table

```php
#[Entity]
class Commentable
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $type;
}

#[Entity]
class Comment
{
    #[BelongsTo(target: Commentable::class)]
    private Commentable $commentable;
}
```

**Pros:** Foreign keys, normalized structure
**Cons:** Extra entity, more complex to manage

## See Also

- [BelongsTo Relations](/docs/en/relation/belongs-to.md) - Standard parent-child relationships
- [HasMany Relations](/docs/en/relation/has-many.md) - One-to-many relationships
- [Relation Collections](/docs/en/relation/collections.md) - Collection management
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced querying
