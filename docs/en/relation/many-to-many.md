# Many To Many

The **Many To Many** relation connects two entities through an intermediate pivot (junction) table. This is the standard
way to model many-to-many relationships in relational databases.

**Examples:** Users have many tags, posts have many categories, students enroll in many courses.

> Many To Many is actually two HasMany relations combined: source → pivot and pivot → target.

## Definition

To define a ManyToMany relation, you need three entities: source, target, and pivot.

### Complete Example

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\ManyToMany;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $username;

    #[ManyToMany(target: Tag::class, through: UserTag::class)]
    private array $tags = [];

    public function getTags(): array
    {
        return $this->tags;
    }

    public function addTag(Tag $tag): void
    {
        $this->tags[] = $tag;
    }

    public function removeTag(Tag $tag): void
    {
        $this->tags = array_filter(
            $this->tags,
            static fn(Tag $t) => $t !== $tag
        );
    }
}

#[Entity]
class Tag
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;
}
```

### Attribute Specification

| Parameter       | Type          | Default   | Description                                                                                 |
|-----------------|---------------|-----------|---------------------------------------------------------------------------------------------|
| target          | string        | -         | **Required**. Target entity role or class name                                              |
| through         | string        | -         | **Required**. Pivot entity role or class name                                               |
| load            | string        | 'lazy'    | Loading strategy: 'lazy' or 'eager'                                                         |
| cascade         | bool          | true      | Automatically save related entities with source entity                                      |
| nullable        | bool          | false     | Whether relations can be null (affects FK constraints in pivot)                             |
| innerKey        | string\|array | null      | Key column(s) in source entity. Defaults to source's primary key                            |
| outerKey        | string\|array | null      | Key column(s) in target entity. Defaults to target's primary key                            |
| throughInnerKey | string\|array | null      | Foreign key column(s) in pivot referencing source. Defaults to `{sourceRole}_{innerKey}`    |
| throughOuterKey | string\|array | null      | Foreign key column(s) in pivot referencing target. Defaults to `{targetRole}_{outerKey}`    |
| where           | array         | []        | WHERE conditions applied to target entity when loading                                      |
| throughWhere    | array         | []        | WHERE conditions applied to pivot entity when loading                                       |
| orderBy         | array         | []        | Default sorting for loaded collection                                                       |
| fkCreate        | bool          | true      | Automatically create foreign key constraints in pivot table                                 |
| fkAction        | string        | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'           |
| fkOnDelete      | string        | null      | Foreign key DELETE action (overrides `fkAction` if set): 'CASCADE', 'NO ACTION', 'SET NULL' |
| indexCreate     | bool          | true      | Automatically create index on [throughInnerKey, throughOuterKey]                            |
| collection      | string        | null      | Collection class for loaded entities. See [Collections](#collections)                       |
| inverse         | Inverse       | null      | Configure inverse relation on target entity. See [Inverse Relations](#inverse-relations)    |

> Since Cycle ORM v2.x, all key parameters can be arrays for [composite keys](/docs/en/advanced/composite-pk.md).

### Key Behavior

By default, the ORM generates foreign key columns in the pivot table:

```php
#[Entity]
class User
{
    #[ManyToMany(target: Tag::class, through: UserTag::class)]
    private array $tags = [];
}
```

This creates in the `UserTag` pivot table:

- `user_id` → references `user.id`
- `tag_id` → references `tag.id`

Customize column names:

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    throughInnerKey: 'person_id',
    throughOuterKey: 'label_id'
)]
private array $tags = [];
```

### Pivot Entity

The pivot entity can be minimal (just a primary key) or contain additional data:

#### Minimal Pivot

```php
#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;
}
```

#### Pivot with Additional Data

```php
#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'datetime')]
    private \DateTimeInterface $assignedAt;

    #[Column(type: 'integer')]
    private int $priority = 0;

    public function __construct()
    {
        $this->assignedAt = new \DateTimeImmutable();
    }

    public function getAssignedAt(): \DateTimeInterface
    {
        return $this->assignedAt;
    }

    public function setPriority(int $priority): void
    {
        $this->priority = $priority;
    }
}
```

## Usage Examples

### Creating Relations

Related entities are automatically saved with the source (unless `cascade: false`):

```php
$user = new User();
$user->setUsername("johndoe");

$tag1 = new Tag("php");
$tag2 = new Tag("database");
$tag3 = new Tag("orm");

$user->addTag($tag1);
$user->addTag($tag2);
$user->addTag($tag3);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

The save order:

1. Source entity (`User`) is saved
2. Target entities (`Tag`) are saved
3. Pivot records (`UserTag`) are created linking source and targets

### Managing Collections

#### Adding to Existing Collection

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

$newTag = new Tag("security");
$user->addTag($newTag);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

#### Checking Existence

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

$tag = $orm->getRepository(Tag::class)->findOne(['name' => 'php']);

if (!in_array($tag, $user->getTags(), true)) {
    $user->addTag($tag);
}
```

### Removing Relations

Removing from the collection deletes the pivot record, **not the target entity**:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

$tagToRemove = $user->getTags()[0];
$user->removeTag($tagToRemove);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
// UserTag pivot record deleted, Tag entity remains
```

#### Clearing All Relations

```php
$user->setTags([]);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
// All UserTag pivot records deleted
```

## Loading

### Basic Loading

Load related entities explicitly:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

foreach ($user->getTags() as $tag) {
    echo $tag->getName() . "\n";
}
```

### Eager Loading

Configure automatic loading:

```php
#[ManyToMany(target: Tag::class, through: UserTag::class, load: 'eager')]
private array $tags = [];
```

```php
$user = $orm->getRepository(User::class)->findByPK(1);
// Tags are already loaded
print_r($user->getTags());
```

### Filtered Loading

Pre-filter target entities when loading:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags', ['where' => ['active' => true]])
    ->fetchAll();
```

Set default filters in the relation:

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    where: ['active' => true, 'verified' => true]
)]
private array $tags = [];
```

### Sorted Loading

Specify sorting when loading:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags', ['orderBy' => ['name' => 'ASC']])
    ->fetchAll();
```

Set default sorting in the relation:

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    orderBy: ['priority' => 'DESC', 'name' => 'ASC']
)]
private array $tags = [];
```

## Filtering

### Using `with()` for Filtering

Filter source entities based on target entity criteria:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->with('tags')->where('tags.name', 'php')
    ->fetchAll();
```

> **Important:** Always use `distinct()` with ManyToMany filtering to avoid duplicate source rows.

### Automatic Joins

```php
// Automatically joins tags and user_tags tables
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.name', 'php')
    ->where('tags.active', true)
    ->fetchAll();
```

### Complex Filtering

Find users with specific tags:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.name', 'in', ['php', 'database', 'orm'])
    ->having('COUNT(DISTINCT tags.id)', '>=', 2)
    ->fetchAll();
```

### Pivot Filtering

Access pivot table data using the `@` syntax:

```php
// Find users who were assigned tags in the last hour
$hour = new \DateInterval("PT1H");
$recentUsers = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.assigned_at', '>', (new \DateTimeImmutable())->sub($hour))
    ->fetchAll();
```

Filter by pivot relations:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.assignedBy.role', 'admin')
    ->fetchAll();
```

## Pivot Entity Access

### Pivot Data

To access pivot entity data, use the Doctrine collection with pivoted support:

```php
use Cycle\ORM\Collection\Pivoted\PivotedCollection;

#[Entity]
class User
{
    #[ManyToMany(
        target: Tag::class,
        through: UserTag::class,
        collection: 'doctrine'
    )]
    private PivotedCollection $tags;

    public function __construct()
    {
        $this->tags = new PivotedCollection();
    }
}
```

Access pivot data:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

foreach ($user->tags as $tag) {
    $pivot = $user->tags->getPivot($tag);
    echo sprintf(
        "Tag '%s' assigned at %s with priority %d\n",
        $tag->getName(),
        $pivot->getAssignedAt()->format('Y-m-d H:i'),
        $pivot->getPriority()
    );
}
```

### Setting Pivot Data

Create associations with custom pivot data:

```php
$user = new User();
$user->setUsername("johndoe");

$tag = new Tag("php");

$pivot = new UserTag();
$pivot->setPriority(10);

$user->tags->add($tag);
$user->tags->setPivot($tag, $pivot);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

### Pivot Relations

Pivot entities can have their own relations:

```php
#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;

    #[BelongsTo(target: User::class)]
    private User $assignedBy;

    public function __construct(User $assignedBy)
    {
        $this->assignedBy = $assignedBy;
        $this->assignedAt = new \DateTimeImmutable();
    }
}
```

Load pivot relations:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags.@.assignedBy') // Load pivot's assignedBy relation
    ->fetchAll();
```

## Complex Loading

### Sorting by Pivot Data

Load with pivot-based sorting:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags', [
        'load' => function (\Cycle\ORM\Select\QueryBuilder $q) {
            // @ = current relation, @.@ = pivot entity
            $q->orderBy('@.@.priority', 'DESC');
        }
    ])
    ->fetchAll();
```

### Single Query Loading

Force single-query loading for complex scenarios:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags', [
        'method' => \Cycle\ORM\Select::SINGLE_QUERY,
        'load' => function (\Cycle\ORM\Select\QueryBuilder $q) {
            $q->where('@.@.priority', '>', 5)
              ->orderBy('@.@.priority', 'DESC');
        }
    ])
    ->fetchAll();
```

### Combined Filtering and Loading

Filter source, then load with pivot conditions:

```php
$activeUsers = $orm->getRepository(User::class)
    ->select()
    ->where('active', true)
    ->load('tags', [
        'where' => ['verified' => true],
        'load' => function (\Cycle\ORM\Select\QueryBuilder $q) {
            $q->where('@.@.priority', '>=', 5)
              ->orderBy('@.@.priority', 'DESC')
              ->orderBy('name', 'ASC');
        }
    ])
    ->fetchAll();
```

## Collections

ManyToMany supports different collection types:

### Default Array Collection

```php
#[ManyToMany(target: Tag::class, through: UserTag::class)]
private array $tags = [];
```

### Doctrine Collection (with Pivot Access)

```php
use Doctrine\Common\Collections\Collection;
use Cycle\ORM\Collection\Pivoted\PivotedCollection;

#[Entity]
class User
{
    #[ManyToMany(
        target: Tag::class,
        through: UserTag::class,
        collection: 'doctrine'
    )]
    private PivotedCollection $tags;

    public function __construct()
    {
        $this->tags = new PivotedCollection();
    }
}
```

### Custom Collection Factory

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    collection: 'my_custom_factory'
)]
private CustomCollection $tags;
```

> Read more about [relation collections](/docs/en/relation/collections.md).

## Inverse Relations

Define bidirectional relationships:

```php
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class User
{
    #[ManyToMany(
        target: Tag::class,
        through: UserTag::class,
        inverse: new Inverse(as: 'users', type: 'manyToMany')
    )]
    private array $tags = [];
}
```

This automatically creates the inverse relation on Tag:

```php
// Equivalent to defining on Tag:
#[Entity]
class Tag
{
    #[ManyToMany(target: User::class, through: UserTag::class)]
    private array $users = [];
}
```

## Foreign Key Options

### Default Behavior

By default, ManyToMany creates foreign keys in the pivot table with CASCADE:

```php
#[ManyToMany(target: Tag::class, through: UserTag::class)]
private array $tags = [];
// Creates in UserTag:
// FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
// FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
```

When either User or Tag is deleted, pivot records are automatically removed.

### Custom Foreign Key Actions

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    fkAction: 'CASCADE',        // Both DELETE and UPDATE
)]
private array $tags = [];
```

### Separate DELETE Action

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    fkAction: 'CASCADE',
    fkOnDelete: 'SET NULL',
    nullable: true
)]
private array $tags = [];
```

**Available Actions:**

| Action    | Description                                                 |
|-----------|-------------------------------------------------------------|
| CASCADE   | Delete pivot records when source or target is deleted       |
| SET NULL  | Set FK to NULL (requires `nullable: true`)                  |
| NO ACTION | Prevent deletion if pivot records exist (database enforced) |

### Disable Foreign Key Creation

```php
#[ManyToMany(
    target: Tag::class,
    through: UserTag::class,
    fkCreate: false,
    indexCreate: false
)]
private array $tags = [];
```

## Best Practices

1. **Always initialize collections** to empty arrays in property declarations
2. **Use distinct()** when filtering by target properties
3. **Keep pivot entities simple** unless you need additional data
4. **Use Doctrine collections** when you need pivot access
5. **Index pivot foreign keys** for query performance (enabled by default)
6. **Consider cascade behavior** - pivot records should usually cascade delete
7. **Avoid cross-database** ManyToMany relations (not supported)

## Common Patterns

### Tags/Categories System

```php
#[Entity]
class Post
{
    #[ManyToMany(
        target: Tag::class,
        through: PostTag::class,
        orderBy: ['name' => 'ASC']
    )]
    private array $tags = [];

    public function hasTag(string $name): bool
    {
        foreach ($this->tags as $tag) {
            if ($tag->getName() === $name) {
                return true;
            }
        }
        return false;
    }
}
```

### Timestamped Relations

```php
#[Entity]
class UserTag
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'datetime')]
    private \DateTimeInterface $createdAt;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }
}
```

```php
// Find recent associations
$users = $orm->getRepository(User::class)
    ->select()
    ->distinct()
    ->where('tags.@.created_at', '>', new DateTime('-7 days'))
    ->fetchAll();
```

### Weighted Relations

```php
#[Entity]
class UserSkill
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'integer')]
    private int $level; // 1-10

    public function setLevel(int $level): void
    {
        if ($level < 1 || $level > 10) {
            throw new \InvalidArgumentException('Level must be 1-10');
        }
        $this->level = $level;
    }
}
```

### Preventing Duplicates

```php
public function addTag(Tag $tag): void
{
    foreach ($this->tags as $existingTag) {
        if ($existingTag->getId() === $tag->getId()) {
            return; // Already exists
        }
    }
    $this->tags[] = $tag;
}
```

## Performance Considerations

### Loading Strategy

```php
// Separate queries (default) - better for many relations
$user = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->wherePK(1)
    ->fetchOne();

// Single query with JOINs - better for filtering
$user = $orm->getRepository(User::class)
    ->select()
    ->with('tags')
    ->load('tags', ['using' => 'tags'])
    ->wherePK(1)
    ->fetchOne();
```

### Pagination

For large many-to-many collections, paginate on the target side:

```php
$user = $orm->getRepository(User::class)->findByPK(1);

$tags = $orm->getRepository(Tag::class)
    ->select()
    ->with('users')
    ->where('users.id', $user->getId())
    ->orderBy('name')
    ->limit(20)
    ->offset(0)
    ->fetchAll();
```

### Batch Loading

Load multiple entities with their relations efficiently:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('tags')
    ->where('active', true)
    ->fetchAll();
// Single query for users, single query for all tags
```

## Limitations

- **Cross-database relations** are not currently supported
- **Pivot entity** must have a primary key
- **Composite keys** in pivot table require careful configuration

## See Also

- [HasMany Relations](/docs/en/relation/has-many.md) - One-to-many relationships
- [BelongsTo Relations](/docs/en/relation/belongs-to.md) - Many-to-one relationships
- [Relation Collections](/docs/en/relation/collections.md) - Collection management
- [Composite Keys](/docs/en/advanced/composite-pk.md) - Using composite keys
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced querying
