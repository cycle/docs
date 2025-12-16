# Has One

The **Has One** relation defines that an entity exclusively owns another entity in a parent-child relationship. This
relation is useful for decomposing entities by storing related data in a separate table.

> The parent entity is persisted first, then the child entity with a reference to the parent.

## Table of Contents

- [Definition](#definition)
    - [Attribute Specification](#attribute-specification)
    - [Key Behavior](#key-behavior)
- [Usage Examples](#usage-examples)
    - [Creating Relations](#creating-relations)
    - [Removing Children](#removing-children)
    - [Transferring Children](#transferring-children)
- [Loading](#loading)
- [Filtering](#filtering)
- [Inverse Relations](#inverse-relations)
- [Foreign Key Options](#foreign-key-options)

## Definition

To define a HasOne relation using the annotated entities extension:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\HasOne;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $username;

    #[HasOne(target: Profile::class)]
    private ?Profile $profile = null;

    public function getProfile(): ?Profile
    {
        return $this->profile;
    }

    public function setProfile(?Profile $profile): void
    {
        $this->profile = $profile;
    }
}

#[Entity]
class Profile
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $bio;
}
```

### Attribute Specification

| Parameter   | Type          | Default   | Description                                                                                 |
|-------------|---------------|-----------|---------------------------------------------------------------------------------------------|
| target      | string        | -         | **Required**. Target entity role or class name                                              |
| load        | string        | 'lazy'    | Loading strategy: 'lazy' or 'eager'                                                         |
| cascade     | bool          | true      | Automatically save child entity with parent entity                                          |
| nullable    | bool          | false     | Whether child can exist without parent (affects foreign key NULL constraint)                |
| innerKey    | string\|array | null      | Key column(s) in parent entity. Defaults to parent's primary key                            |
| outerKey    | string\|array | null      | Foreign key column(s) in child entity. Defaults to `{parentRole}_{innerKey}`                |
| fkCreate    | bool          | true      | Automatically create foreign key constraint                                                 |
| fkAction    | string        | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'           |
| fkOnDelete  | string        | null      | Foreign key DELETE action (overrides `fkAction` if set): 'CASCADE', 'NO ACTION', 'SET NULL' |
| indexCreate | bool          | true      | Automatically create index on foreign key column(s)                                         |
| inverse     | Inverse       | null      | Configure inverse relation on child entity. See [Inverse Relations](#inverse-relations)     |

> Since Cycle ORM v2.x, `innerKey` and `outerKey` can be arrays for [composite keys](/docs/en/advanced/composite-pk.md).

### Key Behavior

By default, the ORM generates a foreign key column in the child entity using the pattern
`{parentRole}_{parentPrimaryKey}`:

```php
#[Entity]
class User
{
    // Creates column in Profile: user_id (references user.id)
    #[HasOne(target: Profile::class)]
    private ?Profile $profile;
}
```

Customize the foreign key column:

```php
#[Entity]
class User
{
    // Uses custom column in Profile: owner_id
    #[HasOne(target: Profile::class, outerKey: 'owner_id')]
    private ?Profile $profile;
}
```

## Usage Examples

### Creating Relations

The child entity is automatically saved with the parent when persisted (unless `cascade: false`):

```php
$user = new User();
$user->setUsername("johndoe");

$profile = new Profile();
$profile->setBio("Software Developer");

$user->setProfile($profile);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

The save order ensures:

1. Parent (`User`) is saved first, generating its ID
2. Child (`Profile`) is saved second, with parent's ID as the foreign key

### Removing Children

To delete the child entity, set the relation to `null`:

```php
$user = $orm->getRepository(User::class)->findByPK(1);
$user->setProfile(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run(); // Profile is deleted
```

#### Detaching Instead of Deleting

If you want to detach the child without deleting it, use `nullable: true`:

```php
#[HasOne(target: Profile::class, nullable: true)]
private ?Profile $profile;
```

```php
$user->setProfile(null);
// Profile is detached (user_id set to NULL) but not deleted
```

### Transferring Children

You can transfer a child entity from one parent to another:

```php
$user1 = $orm->getRepository(User::class)
    ->select()
    ->load('profile')
    ->wherePK(1)
    ->fetchOne();

$user2 = new User();
$user2->setUsername("janedoe");

// Transfer profile from user1 to user2
$user2->setProfile($user1->getProfile());
$user1->setProfile(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user1);
$manager->persist($user2);
$manager->run();
```

## Loading

### Explicit Loading

Load the child entity explicitly using the `load()` method:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('profile')
    ->wherePK(1)
    ->fetchOne();

print_r($user->getProfile());
```

### Eager Loading

Configure the relation to always load with the parent:

```php
#[HasOne(target: Profile::class, load: 'eager')]
private ?Profile $profile;
```

With eager loading:

```php
$user = $orm->getRepository(User::class)->findByPK(1);
// Profile is already loaded
print_r($user->getProfile());
```

### Loading Multiple Relations

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->load('profile')
    ->load('settings')
    ->load('preferences')
    ->fetchAll();
```

## Filtering

### Using `with()` for Filtering

Filter parent entities based on child entity criteria:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('profile')->where('profile.verified', true)
    ->fetchAll();
```

### Automatic Joins

The Select query automatically joins related tables when referenced in conditions:

```php
// Automatically joins the profile table
$users = $orm->getRepository(User::class)
    ->select()
    ->where('profile.verified', true)
    ->where('profile.bio', '!=', '')
    ->fetchAll();
```

### Complex Filtering

Combine multiple conditions:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->where('profile.verified', true)
    ->where('profile.country', 'USA')
    ->where('created_at', '>', new DateTime('-1 year'))
    ->orderBy('profile.reputation', 'DESC')
    ->fetchAll();
```

### Filtering on NULL Relations

Find users without profiles:

```php
$usersWithoutProfiles = $orm->getRepository(User::class)
    ->select()
    ->where('profile.id', null)
    ->fetchAll();
```

## Inverse Relations

Define bidirectional relationships using the `inverse` parameter:

```php
use Cycle\Annotated\Annotation\Relation\Inverse;

#[Entity]
class User
{
    #[HasOne(
        target: Profile::class,
        inverse: new Inverse(as: 'user', type: 'belongsTo')
    )]
    private ?Profile $profile;
}
```

This automatically creates the inverse BelongsTo relation on Profile:

```php
// Equivalent to defining on Profile:
#[Entity]
class Profile
{
    #[BelongsTo(target: User::class)]
    private User $user;
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

By default, HasOne creates a foreign key with CASCADE on both DELETE and UPDATE:

```php
#[HasOne(target: Profile::class)]
private ?Profile $profile;
// Creates: FOREIGN KEY (user_id) REFERENCES users(id) 
//          ON DELETE CASCADE ON UPDATE CASCADE
```

When a user is deleted, their profile is automatically deleted.

### Custom Foreign Key Actions

Control what happens when the parent is deleted or updated:

```php
#[HasOne(
    target: Profile::class,
    fkAction: 'SET NULL',      // Both DELETE and UPDATE
    nullable: true              // Required for SET NULL
)]
private ?Profile $profile;
```

### Separate DELETE Action

Set different behavior for DELETE operations:

```php
#[HasOne(
    target: Profile::class,
    fkAction: 'CASCADE',        // Default for UPDATE
    fkOnDelete: 'NO ACTION',    // Prevent deletion if profile exists
)]
private ?Profile $profile;
```

**Available Actions:**

| Action    | Description                                                        |
|-----------|--------------------------------------------------------------------|
| CASCADE   | Delete/update child when parent is deleted/updated                 |
| SET NULL  | Set foreign key to NULL (requires `nullable: true`)                |
| NO ACTION | Prevent parent deletion/update if child exists (database enforced) |

### Disable Foreign Key Creation

For manual control or cross-database relations:

```php
#[HasOne(
    target: Profile::class,
    fkCreate: false,
    indexCreate: false
)]
private ?Profile $profile;
```

### Manual Column Definition

Explicitly define the foreign key column in the child entity:

```php
#[Entity]
class Profile
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'integer', nullable: false)]
    private int $user_id;

    #[Column(type: 'string')]
    private string $bio;
}
```

```php
#[Entity]
class User
{
    #[HasOne(target: Profile::class, outerKey: 'user_id')]
    private ?Profile $profile;
}
```

## Best Practices

1. **Initialize optional relations to null** in property declarations
2. **Use cascade deletion carefully** - ensure you want the child deleted with the parent
3. **Consider nullable relations** when children can exist independently
4. **Use explicit loading** for better performance control
5. **Validate before setting** - ensure the child entity is valid before assignment

## Common Patterns

### Lazy Initialization

```php
#[Entity]
class User
{
    #[HasOne(target: Profile::class)]
    private ?Profile $profile = null;

    public function getProfile(): Profile
    {
        if ($this->profile === null) {
            $this->profile = new Profile();
        }
        return $this->profile;
    }
}
```

### Optional Relations with Defaults

```php
#[Entity]
class User
{
    #[HasOne(target: Settings::class, nullable: true)]
    private ?Settings $settings = null;

    public function getSettings(): Settings
    {
        return $this->settings ?? Settings::getDefaults();
    }
}
```

### Conditional Child Creation

```php
#[Entity]
class User
{
    #[HasOne(target: Profile::class, nullable: true)]
    private ?Profile $profile = null;

    public function activateProfile(string $bio): void
    {
        if ($this->profile === null) {
            $this->profile = new Profile();
        }
        $this->profile->setBio($bio);
        $this->profile->setActive(true);
    }
}
```

## Differences from BelongsTo

| Aspect           | HasOne                              | BelongsTo                           |
|------------------|-------------------------------------|-------------------------------------|
| Foreign Key      | Stored in child entity              | Stored in child entity              |
| Save Order       | Parent first, then child            | Parent first, then child            |
| Ownership        | Parent owns child                   | Child references parent             |
| Default Cascade  | true                                | true                                |
| Typical Use Case | One-to-one ownership (user→profile) | Many-to-one reference (post→author) |
| Delete Behavior  | Child deleted when parent deleted   | Child references parent via FK      |

## See Also

- [BelongsTo Relations](/docs/en/relation/belongs-to.md) - The inverse of HasOne
- [HasMany Relations](/docs/en/relation/has-many.md) - One-to-many relationships
- [Embedded Relations](/docs/en/relation/embedded.md) - Alternative to HasOne for same-table storage
- [Composite Keys](/docs/en/advanced/composite-pk.md) - Using composite keys in relations
- [Query Builder](/docs/en/query-builder/relations.md) - Advanced relation querying
