# Relations

The Cycle Annotated package provides multiple attributes designed to describe entity relations. Each relation must be
associated with specific entity properties in order to work. In addition, most of the relation options (such as the name
of inner, outer keys) will be generated automatically.

> You can read more about relation configuration and usage in the dedicated relation documentation pages linked below.

## Table of Contents

- [Common Configuration](#common-configuration)
- [Relation Types](#relation-types)
    - [Embedded](#embedded)
    - [BelongsTo](#belongsto)
    - [RefersTo](#refersto)
    - [HasOne](#hasone)
    - [HasMany](#hasmany)
    - [ManyToMany](#manytomany)
    - [Morphed Relations](#morphed-relations)
- [Inverse Relations](#inverse-relations)
- [Foreign Key Configuration](#foreign-key-configuration)
- [Loading Strategies](#loading-strategies)

## Common Configuration

Each relation must have a proper `target` option. The target must point to either the related entity `role`, or to the
class name. You are able to specify class names in a fully qualified (Namespace\Class) or using current entity namespace
as the base path. You can use both `/` and `\` namespace separators.

### Common Relation Parameters

Most relation types share these common parameters:

| Parameter | Type          | Default | Description                                                                         |
|-----------|---------------|---------|-------------------------------------------------------------------------------------|
| target    | string        | -       | **Required**. Target entity role or class name                                      |
| load      | string        | 'lazy'  | Loading strategy: 'lazy' or 'eager'. Read more about [loading](#loading-strategies) |
| cascade   | bool          | true    | Automatically persist related entities with parent entity                           |
| nullable  | bool          | varies  | Whether relation can be null (varies by relation type)                              |
| innerKey  | string\|array | null    | Key(s) in source entity. Defaults vary by relation type                             |
| outerKey  | string\|array | null    | Key(s) in target entity. Defaults vary by relation type                             |

### Foreign Key Parameters

Relations that create foreign keys support these additional parameters:

| Parameter   | Type   | Default   | Description                                                                       |
|-------------|--------|-----------|-----------------------------------------------------------------------------------|
| fkCreate    | bool   | true      | Automatically create foreign key constraint                                       |
| fkAction    | string | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL' |
| fkOnDelete  | string | null      | Specific FK onDelete action. Overrides `fkAction` if set                          |
| indexCreate | bool   | true      | Automatically create index on foreign key columns                                 |

> Read more about [foreign key configuration](#foreign-key-configuration) below.

## Relation Types

### Embedded

Embeds another entity's columns into the parent table. Embedded entities are always loaded with the parent.

**Documentation:** [Embedded Relations](/docs/en/relation/embedded.md)

```php
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Embedded(target: Address::class)]
private Address $address;
```

**Key Parameters:**

| Parameter | Type   | Default | Description                               |
|-----------|--------|---------|-------------------------------------------|
| target    | string | -       | **Required**. Embeddable entity class     |
| load      | string | 'eager' | Loading strategy ('eager' or 'lazy')      |
| prefix    | string | null    | Column prefix for embedded entity columns |

> Read more about [Embeddable entities](/docs/en/annotated/embeddings.md).

### BelongsTo

Defines that an entity belongs to a parent entity. The child stores the foreign key.

**Documentation:** [BelongsTo Relations](/docs/en/relation/belongs-to.md)

```php
use Cycle\Annotated\Annotation\Relation\BelongsTo;

#[BelongsTo(target: User::class)]
private User $author;
```

**Key Parameters:**

| Parameter | Type          | Default | Description                                               |
|-----------|---------------|---------|-----------------------------------------------------------|
| target    | string        | -       | **Required**. Parent entity                               |
| innerKey  | string\|array | null    | Foreign key in child. Defaults to `{relation}_{outerKey}` |
| outerKey  | string\|array | null    | Key in parent. Defaults to primary key                    |
| nullable  | bool          | false   | Can child exist without parent                            |
| cascade   | bool          | true    | Auto-save parent with child                               |

> Supports [foreign key parameters](#foreign-key-parameters).

### RefersTo

Similar to BelongsTo but used for multiple relations to the same entity or cyclic references.

**Documentation:** [RefersTo Relations](/docs/en/relation/refers-to.md)

```php
use Cycle\Annotated\Annotation\Relation\RefersTo;

#[RefersTo(target: Comment::class)]
private ?Comment $lastComment;
```

**Key Parameters:**

| Parameter | Type          | Default | Description                                                |
|-----------|---------------|---------|------------------------------------------------------------|
| target    | string        | -       | **Required**. Target entity                                |
| innerKey  | string\|array | null    | Key in source entity                                       |
| outerKey  | string\|array | null    | Foreign key in target. Defaults to `{relation}_{innerKey}` |
| nullable  | bool          | false   | Can reference be null                                      |

> Entity is persisted before related entity, then updated. Supports [foreign key parameters](#foreign-key-parameters).

### HasOne

Defines a one-to-one relationship where the child entity stores the foreign key.

**Documentation:** [HasOne Relations](/docs/en/relation/has-one.md)

```php
use Cycle\Annotated\Annotation\Relation\HasOne;

#[HasOne(target: Profile::class)]
private ?Profile $profile;
```

**Key Parameters:**

| Parameter | Type          | Default | Description                                             |
|-----------|---------------|---------|---------------------------------------------------------|
| target    | string        | -       | **Required**. Child entity                              |
| innerKey  | string\|array | null    | Key in parent. Defaults to primary key                  |
| outerKey  | string\|array | null    | Foreign key in child. Defaults to `{parent}_{innerKey}` |
| nullable  | bool          | false   | Can child exist without parent                          |
| cascade   | bool          | true    | Auto-save child with parent                             |

> Supports [foreign key parameters](#foreign-key-parameters).

### HasMany

Defines a one-to-many relationship where children store the foreign key.

**Documentation:** [HasMany Relations](/docs/en/relation/has-many.md)

```php
use Cycle\Annotated\Annotation\Relation\HasMany;

#[HasMany(target: Post::class)]
private array $posts = [];
```

**Key Parameters:**

| Parameter  | Type          | Default | Description                                                                                      |
|------------|---------------|---------|--------------------------------------------------------------------------------------------------|
| target     | string        | -       | **Required**. Child entity                                                                       |
| innerKey   | string\|array | null    | Key in parent. Defaults to primary key                                                           |
| outerKey   | string\|array | null    | Foreign key in children. Defaults to `{parent}_{innerKey}`                                       |
| where      | array         | []      | Additional WHERE conditions for loading                                                          |
| orderBy    | array         | []      | Sorting rules for loaded collection                                                              |
| collection | string        | null    | Collection class for loaded entities. Read about [collections](/docs/en/relation/collections.md) |

> Supports [foreign key parameters](#foreign-key-parameters).

### ManyToMany

Defines a many-to-many relationship using a pivot (junction) table.

**Documentation:** [ManyToMany Relations](/docs/en/relation/many-to-many.md)

```php
use Cycle\Annotated\Annotation\Relation\ManyToMany;

#[ManyToMany(target: Tag::class, through: PostTag::class)]
private array $tags = [];
```

**Key Parameters:**

| Parameter       | Type          | Default | Description                                                                  |
|-----------------|---------------|---------|------------------------------------------------------------------------------|
| target          | string        | -       | **Required**. Target entity                                                  |
| through         | string        | -       | **Required**. Pivot entity class                                             |
| innerKey        | string\|array | null    | Key in source. Defaults to primary key                                       |
| outerKey        | string\|array | null    | Key in target. Defaults to primary key                                       |
| throughInnerKey | string\|array | null    | FK in pivot to source. Defaults to `{source}_{innerKey}`                     |
| throughOuterKey | string\|array | null    | FK in pivot to target. Defaults to `{target}_{outerKey}`                     |
| where           | array         | []      | WHERE conditions for target entity                                           |
| throughWhere    | array         | []      | WHERE conditions for pivot entity                                            |
| orderBy         | array         | []      | Sorting rules                                                                |
| collection      | string        | null    | Collection class. Read about [collections](/docs/en/relation/collections.md) |

> Supports [foreign key parameters](#foreign-key-parameters) for pivot table constraints.

### Morphed Relations

Polymorphic relations allow an entity to belong to multiple entity types. These relations use an interface as target.

**Documentation:** [Morphed Relations](/docs/en/relation/morphed.md)

#### BelongsToMorphed

Child belongs to any parent implementing an interface.

```php
use Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed;

#[BelongsToMorphed(target: ImageHolderInterface::class)]
private ImageHolderInterface $owner;
```

**Key Parameters:**

| Parameter      | Type          | Default | Description                                               |
|----------------|---------------|---------|-----------------------------------------------------------|
| target         | string        | -       | **Required**. Target interface                            |
| innerKey       | string\|array | null    | FK in child. Defaults to `{relation}_{outerKey}`          |
| outerKey       | string\|array | null    | Key in parent. Defaults to primary key                    |
| morphKey       | string        | null    | Column storing parent type. Defaults to `{relation}_role` |
| morphKeyLength | int           | 32      | Length of morph key column                                |
| indexCreate    | bool          | true    | Create index on [morphKey, innerKey]                      |

#### MorphedHasOne

Parent owns one child through polymorphic relation.

```php
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasOne;

#[MorphedHasOne(target: Image::class)]
private ?Image $avatar;
```

**Key Parameters:**

| Parameter      | Type          | Default | Description                                    |
|----------------|---------------|---------|------------------------------------------------|
| target         | string        | -       | **Required**. Child entity                     |
| innerKey       | string\|array | null    | Key in parent. Defaults to primary key         |
| outerKey       | string\|array | null    | FK in child. Defaults to `{parent}_{innerKey}` |
| morphKey       | string        | null    | Column in child storing parent type            |
| morphKeyLength | int           | 32      | Length of morph key column                     |

#### MorphedHasMany

Parent owns many children through polymorphic relation.

```php
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasMany;

#[MorphedHasMany(target: Comment::class)]
private array $comments = [];
```

**Key Parameters:**

| Parameter      | Type          | Default | Description                                       |
|----------------|---------------|---------|---------------------------------------------------|
| target         | string        | -       | **Required**. Child entity                        |
| innerKey       | string\|array | null    | Key in parent. Defaults to primary key            |
| outerKey       | string\|array | null    | FK in children. Defaults to `{parent}_{innerKey}` |
| morphKey       | string        | null    | Column in children storing parent type            |
| morphKeyLength | int           | 32      | Length of morph key column                        |
| where          | array         | []      | Additional WHERE conditions                       |
| collection     | string        | null    | Collection class                                  |

> **Note:** Morphed relations do not support foreign key constraints since the target can be multiple entity types.

## Inverse Relations

Many relation types support inverse (bidirectional) relations using the `#[Inverse]` attribute or `inverse` parameter:

```php
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class User
{
    #[HasMany(target: Post::class, inverse: new Inverse(as: 'author', type: 'belongsTo'))]
    private array $posts = [];
}

// Equivalent to defining on the Post entity:
#[Entity]
class Post
{
    #[BelongsTo(target: User::class)]
    private User $author;
}
```

**Inverse Attribute Parameters:**

| Parameter | Type   | Description                                                         |
|-----------|--------|---------------------------------------------------------------------|
| as        | string | **Required**. Property name for inverse relation in target          |
| type      | string | **Required**. Relation type: 'belongsTo', 'hasOne', 'hasMany', etc. |
| load      | string | Loading strategy for inverse relation: 'eager', 'lazy', 'promise'   |

> Read more about inverse relations in individual relation documentation pages.

## Foreign Key Configuration

Most relations automatically create foreign key constraints. You can control this behavior:

### Default Behavior

```php
#[BelongsTo(target: User::class)]
private User $user;
// Creates FK: post.user_id -> user.id with CASCADE
```

### Custom FK Actions

```php
#[BelongsTo(
    target: User::class,
    fkAction: 'SET NULL',        // Both DELETE and UPDATE
    fkOnDelete: 'CASCADE'        // Overrides fkAction for DELETE only
)]
private ?User $user;
```

### Disable FK Creation

```php
#[HasMany(target: Post::class, fkCreate: false)]
private array $posts = [];
```

### Disable Index Creation

```php
#[BelongsTo(target: User::class, indexCreate: false)]
private User $user;
```

## Loading Strategies

Relations support two primary loading strategies:

### Lazy Loading (Default)

Relations are loaded on-demand when accessed:

```php
#[BelongsTo(target: User::class, load: 'lazy')]
private User $author;

// Query to load post
$post = $orm->getRepository(Post::class)->findByPK(1);
// Separate query when accessing author
$author = $post->getAuthor();
```

### Eager Loading

Relations are loaded immediately with parent entity:

```php
#[BelongsTo(target: User::class, load: 'eager')]
private User $author;

// Single query loads both post and author
$post = $orm->getRepository(Post::class)->findByPK(1);
```

> **Best Practice:** Use lazy loading by default and explicitly load relations when needed:
> ```php
> $posts = $orm->getRepository(Post::class)->select()->load('author')->fetchAll();
> ```

> Read more about [loading strategies in Query Builder](/docs/en/query-builder/relations.md).

## See Also

- [Embeddable Entities](/docs/en/annotated/embeddings.md)
- [Composite Primary Keys](/docs/en/advanced/composite-pk.md)
- [Relation Collections](/docs/en/relation/collections.md)
- [Query Builder - Relations](/docs/en/query-builder/relations.md)
