# Annotated Entities

The annotated entities' extension is capable of indexing any domain entity in your project. To indicate that the class
must be treated as a domain entity make sure to add the `#[Entity]` attribute.

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User
{
    // ...
}
```

> Read more about [Prerequisites and Setup](/docs/en/annotated/prerequisites.md) for configuring the annotation compiler
> pipeline.

## Table of Contents

- [Entity Attribute](#entity-attribute)
    - [Attribute Specification](#entity-attribute-specification)
    - [Basic Examples](#entity-basic-examples)
- [Column Attributes](#column-attributes)
    - [Column Specification](#column-attribute-specification)
    - [Column Types Reference](#column-types-reference)
    - [Enum Types](#enum-types)
    - [Database-Specific Types](#database-specific-types)
- [Generated Values](#generated-values)
- [Table Extension](#table-extension)
    - [Class-Level Columns](#class-level-columns)
    - [Indexes](#indexes)
    - [Primary Keys](#primary-keys)
- [Foreign Keys](#foreign-keys)
- [Merging Attributes](#merging-attributes)

## Entity Attribute

Usually, the single attribute `#[Entity]` is enough to describe your model. In this case, Cycle ORM will automatically
assign the generated table name and role based on the class name. In the case of `User` the role will be `user`,
database `null` (default) and table `users`.

You can tweak all of these values by setting `entity` options:

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity(role: 'user', database: 'database', table: 'user_table')]
class User
{
    // ...
}
```

> You must manually set `role` and `table` for your classes if you use models that share the same name.

Some options can be used to overwrite default entity behaviour, for example to assign a custom entity repository:

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity(repository: Repository\UserRepository::class)]
class User
{
    // ...
}
```

> Cycle ORM can locate repository class names automatically, using current entity namespace as the base path.

### Entity Attribute Specification

| Parameter      | Type                   | Default | Description                                                                                                                                                           |
|----------------|------------------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| role           | ?string                | null    | Entity role. Defaults to the lowercase class name without a namespace                                                                                                 |
| mapper         | ?class-string          | null    | Mapper class name. Defaults to `Cycle\ORM\Mapper\Mapper`                                                                                                              |
| repository     | ?class-string          | null    | Repository class to represent read operations for an entity. Defaults to `Cycle\ORM\Select\Repository`                                                                |
| table          | ?string                | null    | Entity source table. Defaults to plural form of entity role                                                                                                           |
| readonlySchema | bool                   | false   | Set to `true` to disable schema synchronization for the assigned table                                                                                                |
| database       | ?string                | null    | Database name. Defaults to default database                                                                                                                           |
| source         | ?class-string          | null    | Entity source class (internal). Defaults to `Cycle\ORM\Select\Source`                                                                                                 |
| typecast       | string\|string[]\|null | null    | Typecast handler class name or array of handler class names. Defaults to `Cycle\ORM\Parser\Typecast`. Read more about [typecasting](/docs/en/advanced/typecasting.md) |
| scope          | ?class-string          | null    | Class name of constraint to be applied to every entity query. Read more about [scopes](/docs/en/advanced/scope.md)                                                    |
| columns        | Column[]               | []      | Additional entity columns for unmapped properties. See [Class-Level Columns](#class-level-columns)                                                                    |
| foreignKeys    | ForeignKey[]           | []      | Foreign key definitions without relations. See [Foreign Keys](#foreign-keys)                                                                                          |

### Entity Basic Examples

A typical entity description with multiple options:

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity(
    table: 'users',
    repository: \App\Repository\UserRepository::class,
    scope: \App\Scope\SortByID::class,
    typecast: [\App\Typecast\JsonTypecast::class, \Cycle\ORM\Parser\Typecast::class]
)]
class User
{
    // ...
}
```

## Column Attributes

No entity can operate without some properties mapped to table columns. To map your property to the column add the
attribute `#[Column]` to it. It's mandatory to specify the column type. You must always specify only **one auto
incremental** column (`type: 'primary'`), but one or more **non-incremental** primary columns (`primary: true`).

> Read more about [composite keys](/docs/en/advanced/composite-pk.md).

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;
}

#[Entity]
class Pivot
{
    #[Column(type: 'int', primary: true)]
    private int $postId;

    #[Column(type: 'int', primary: true)]
    private int $tagId;
}
```

> Read how to use non-incremental primary keys in the Advanced section:
> - [UUID](/docs/en/entity-behaviors/uuid.md)
> - [Snowflake IDs](/docs/en/entity-behaviors/uuid.md#snowflake-ids)

You can use multiple attributes at the same time with shorter syntax:

```php
use Cycle\Annotated\Annotation as Cycle;

#[Cycle\Entity]
class User
{
    #[Cycle\Column(type: 'primary')]
    private int $id;
}
```

By default, the entity property will be mapped to the column with the same name as the property. You can change it as
follows:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string', name: 'username')]
    private string $login;
}
```

Some column types support additional arguments, such as length, values, etc.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string(32)')]
    private string $login;

    #[Column(type: 'enum(active,disabled)')]
    private string $status;

    #[Column(type: 'decimal(5,5)')]
    private $balance;
}
```

Use the `default` option to specify the default value of the column:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'enum(active,disabled)', default: 'active')]
    private string $status;
}
```

While adding new columns to entities associated with non-empty tables you are required to either specify a default value
or mark the column as nullable:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string(64)', nullable: true)]
    protected ?string $password = null;
}
```

### Column Attribute Specification

| Parameter      | Type                   | Default | Description                                                                                                                                                       |
|----------------|------------------------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type           | string                 | -       | **Required**. Column type with optional arguments. See [Column Types Reference](#column-types-reference)                                                          |
| name           | ?string                | null    | Column name in database. Defaults to the property name                                                                                                            |
| property       | ?string                | null    | Entity property name. Used for virtual columns that don't map to entity properties                                                                                |
| primary        | bool                   | false   | Explicitly mark column as part of primary key                                                                                                                     |
| nullable       | bool                   | false   | Allow NULL values in column                                                                                                                                       |
| default        | mixed                  | null    | Default column value                                                                                                                                              |
| castDefault    | bool                   | false   | Apply typecast to default value                                                                                                                                   |
| typecast       | callable\|string\|null | null    | Typecast rule. Can be callable or string for default handler: "int", "float", "bool", "datetime". Read more about [typecasting](/docs/en/advanced/typecasting.md) |
| readonlySchema | bool                   | false   | Set to `true` to disable schema synchronization for this column                                                                                                   |
| ...$attributes | mixed                  | -       | Database-specific attributes using named parameters. Example: `#[Column('smallInt', unsigned: true, zerofill: true)]`                                             |

### Column Types Reference

#### Standard Column Types

| Type          | Parameters           | Description                                                                                                            |
|---------------|----------------------|------------------------------------------------------------------------------------------------------------------------|
| **primary**   | -                    | Auto-incrementing integer primary key (typically 32-bit). Only one primary column allowed per entity                   |
| bigPrimary    | -                    | Auto-incrementing big integer primary key (typically 64-bit)                                                           |
| boolean       | -                    | Boolean type (stored as tinyint 0/1 in most databases)                                                                 |
| integer       | -                    | Standard integer (typically 32-bit)                                                                                    |
| tinyInteger   | -                    | Small integer (typically 8-bit). Check DBMS for size limits                                                            |
| smallInteger  | -                    | Small integer (typically 16-bit). Check DBMS for size limits                                                           |
| bigInteger    | -                    | Large integer (typically 64-bit)                                                                                       |
| **string**    | [length:255]         | Variable-length string. Ideal for indexed fields like emails and usernames                                             |
| text          | -                    | Large text field. Check DBMS for size limitations                                                                      |
| tinyText      | -                    | Small text field (MySQL specific, same as text in other databases)                                                     |
| longText      | -                    | Very large text field (MySQL specific, same as text in other databases)                                                |
| double        | -                    | Double precision floating-point number                                                                                 |
| float         | -                    | Single precision floating-point number                                                                                 |
| decimal       | precision, [scale:0] | Fixed-precision decimal number. Example: `decimal(10,2)` for currency                                                  |
| datetime      | -                    | Date and time (automatically uses UTC timezone)                                                                        |
| date          | -                    | Date only (automatically uses UTC timezone)                                                                            |
| time          | -                    | Time only                                                                                                              |
| **timestamp** | -                    | Unix timestamp (stored as integer). Automatically converts to UTC. **Prefer `datetime` for general date/time storage** |
| binary        | -                    | Binary data. Check DBMS for size limitations                                                                           |
| tinyBinary    | -                    | Small binary field (MySQL specific)                                                                                    |
| longBinary    | -                    | Large binary field (MySQL specific)                                                                                    |
| json          | -                    | JSON data. Native support in PostgreSQL and MySQL 5.7+, stored as text in other databases                              |
| uuid          | -                    | UUID/GUID field                                                                                                        |
| ulid          | -                    | ULID (Universally Unique Lexicographically Sortable Identifier)                                                        |
| snowflake     | -                    | Snowflake ID (Twitter's distributed unique ID)                                                                         |

### Enum Types

The ORM supports enum columns with explicit value lists. You can define enums in three ways:

#### 1. Inline Enum Values

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'enum(active,disabled,banned)', default: 'active')]
    private string $status;
}
```

#### 2. PHP 8.1+ Backed Enums

You can use PHP's native backed enums with the `values` parameter:

```php
enum UserStatus: string
{
    case Active = 'active';
    case Disabled = 'disabled';
    case Banned = 'banned';
}

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'enum', values: UserStatus::class, default: 'active')]
    private string $status;
    
    // Or pass a specific enum instance
    #[Column(type: 'enum', values: UserStatus::Active)]
    private string $defaultActive;
}
```

#### 3. Array of Values

```php
#[Entity]
class User
{
    #[Column(type: 'enum', values: ['active', 'disabled', 'banned'])]
    private string $status;
    
    // Mix of strings and enum values
    #[Column(type: 'enum', values: [UserStatus::Active, 'custom'])]
    private string $mixedStatus;
}
```

### Database-Specific Types

#### PostgreSQL-Only Types

| Type         | Description                                 |
|--------------|---------------------------------------------|
| smallPrimary | Auto-incrementing small integer primary key |
| timetz       | Time with timezone                          |
| timestamptz  | Timestamp with timezone                     |
| interval     | Time interval                               |
| bit          | Fixed-length bit string                     |
| bitVarying   | Variable-length bit string                  |
| int4range    | Range of integers                           |
| int8range    | Range of big integers                       |
| numrange     | Range of numeric values                     |
| tsrange      | Timestamp range without timezone            |
| tstzrange    | Timestamp range with timezone               |
| daterange    | Date range                                  |
| jsonb        | Binary JSON with indexing support           |
| point        | Geometric point                             |
| line         | Geometric infinite line                     |
| lseg         | Geometric line segment                      |
| box          | Geometric rectangular box                   |
| path         | Geometric path                              |
| polygon      | Geometric polygon                           |
| circle       | Geometric circle                            |
| cidr         | IPv4 or IPv6 network                        |
| inet         | IPv4 or IPv6 host address                   |
| macaddr      | MAC address (6 bytes)                       |
| macaddr8     | MAC address (8 bytes)                       |
| tsvector     | Text search document                        |
| tsquery      | Text search query                           |

#### SQL Server-Only Types

| Type      | Description             |
|-----------|-------------------------|
| datetime2 | High-precision datetime |

#### Database-Specific Attributes

Use named parameters to pass database-specific options:

```php
// MySQL unsigned and zerofill
#[Column(type: 'integer', unsigned: true, zerofill: true)]
private int $count;

// String length with collation
#[Column(type: 'string', length: 100, collation: 'utf8mb4_unicode_ci')]
private string $name;

// Decimal with specific precision
#[Column(type: 'decimal', precision: 10, scale: 2)]
private string $price;
```

## Generated Values

The `#[GeneratedValue]` attribute marks entity fields whose values are automatically generated either by the database or
by the ORM during persistence operations. This is essential for fields like auto-increment IDs, timestamps, UUIDs, and
other automatically managed values.

### When to use it

- The field value is automatically assigned by the database (auto-increment IDs)
- The value is generated in PHP before database operations (UUIDs, timestamps)
- The value should be regenerated on every update (updated_at timestamps)

**Important:** Without `#[GeneratedValue]`, Cycle ORM treats `null` field values as actual NULL values to persist, which
can cause constraint violations on NOT NULL columns. This attribute tells the ORM to skip sending these fields to the
database when they're unset.

### Generation Timing

The attribute supports three generation modes that can be combined:

#### 1. Database-Generated on Insert (`onInsert: true`)

The database generates the value when the row is inserted. The ORM retrieves the generated value after insertion.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\GeneratedValue;

#[Entity]
class User
{
    // Auto-increment primary key
    #[Column(type: 'primary')]
    #[GeneratedValue(onInsert: true)]
    private int $id;
    
    // Database default timestamp (PostgreSQL: DEFAULT NOW())
    #[Column(type: 'datetime', default: 'CURRENT_TIMESTAMP')]
    #[GeneratedValue(onInsert: true)]
    private \DateTimeInterface $registeredAt;
}
```

**Use cases:**

- Auto-increment primary keys (`primary`, `bigPrimary`)
- Database-side DEFAULT expressions
- Database triggers that populate columns

#### 2. PHP-Generated Before Insert (`beforeInsert: true`)

The ORM generates the value in PHP before sending the INSERT query to the database.

```php
#[Entity]
class Document
{
    #[Column(type: 'uuid', primary: true)]
    #[GeneratedValue(beforeInsert: true)]
    private string $id;
    
    #[Column(type: 'datetime')]
    #[GeneratedValue(beforeInsert: true)]
    private \DateTimeInterface $createdAt;
}
```

**Use cases:**

- UUIDs and ULIDs generated in PHP
- Snowflake IDs calculated in application
- Creation timestamps set by ORM
- Custom business logic for initial values

> Read more about UUID generation in [Entity Behaviors: UUID](/docs/en/entity-behaviors/uuid.md).

#### 3. PHP-Generated Before Update (`beforeUpdate: true`)

The ORM regenerates the value in PHP before each UPDATE query.

```php
#[Entity]
class Article
{
    #[Column(type: 'primary')]
    #[GeneratedValue(onInsert: true)]
    private int $id;
    
    #[Column(type: 'datetime')]
    #[GeneratedValue(beforeInsert: true)]
    private \DateTimeInterface $createdAt;
    
    // Automatically updated on every save
    #[Column(type: 'datetime')]
    #[GeneratedValue(beforeInsert: true, beforeUpdate: true)]
    private \DateTimeInterface $updatedAt;
}
```

**Use cases:**

- Update timestamps (`updatedAt`, `modifiedAt`)
- Version numbers incremented on each change
- Computed fields recalculated on save

### Combining Generation Modes

You can combine `beforeInsert` and `beforeUpdate` for fields that need generation on both operations:

```php
#[Entity]
class Post
{
    #[Column(type: 'datetime')]
    #[GeneratedValue(beforeInsert: true, beforeUpdate: true)]
    private \DateTimeInterface $lastModified; // Set on create AND update
    
    #[Column(type: 'datetime')]
    #[GeneratedValue(beforeInsert: true)]
    private \DateTimeInterface $createdAt; // Set only on create
}
```

### GeneratedValue Attribute Specification

| Parameter    | Type | Default | Description                                                                                                                 |
|--------------|------|---------|-----------------------------------------------------------------------------------------------------------------------------|
| beforeInsert | bool | false   | Generate value in PHP before INSERT. The ORM calls value generators before sending the query to database                    |
| onInsert     | bool | false   | Value is generated by database on INSERT (auto-increment, DEFAULT expressions). The ORM retrieves the value after insertion |
| beforeUpdate | bool | false   | Regenerate value in PHP before UPDATE. The ORM calls value generators before sending the query. Ignored during INSERT       |


> **Integration with Entity Behaviors:** The `#[GeneratedValue]` attribute works seamlessly with Cycle's Entity
> Behaviors system. Behaviors like `CreatedAt`, `UpdatedAt`, and `Uuid` automatically set up proper generation logic. Read
> more in [Entity Behaviors](/docs/en/entity-behaviors/timestamps.md).

> **See also:** For implementing custom value generation logic, explore
> the [Advanced: Custom Generators](/docs/en/advanced/custom-generators.md) section.

## Table Extension

In some cases you might want to specify additional table columns and indexes without linking them to entity properties.
Use class-level attributes to define table schema extensions.

### Class-Level Columns

Define columns that don't map to entity properties:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Table\Index;

#[Entity]
#[Column(name: 'created_at', type: 'datetime')]
#[Column(name: 'deleted_at', type: 'datetime', nullable: true)]
#[Index(columns: ['username'], unique: true)]
#[Index(columns: ['name', 'id DESC'])]
class User
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string(32)')]
    protected string $username;

    #[Column(type: 'string')]
    protected string $name;
}
```

> The column definition is identical to property-level `#[Column]` attributes. Use the `property` parameter to map
> class-level columns to entity properties.

### Indexes

Create indexes using the `#[Index]` attribute at class level:

```php
use Cycle\Annotated\Annotation\Table\Index;

// Simple index
#[Index(columns: ['email'])]

// Unique index
#[Index(columns: ['username'], unique: true)]

// Named index
#[Index(columns: ['user_id', 'created_at'], name: 'user_created_idx')]

// Composite index with sort order
#[Index(columns: ['name ASC', 'id DESC'])]
```

#### Index Attribute Specification

| Parameter | Type     | Default | Description                                 |
|-----------|----------|---------|---------------------------------------------|
| columns   | string[] | -       | **Required**. Array of column names         |
| unique    | bool     | false   | Create unique index                         |
| name      | ?string  | null    | Index name. Auto-generated if not specified |

### Primary Keys

For composite primary keys, use the `#[Table\PrimaryKey]` attribute:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Table\PrimaryKey;

#[Entity]
#[PrimaryKey(columns: ['user_id', 'post_id'])]
class UserPost
{
    #[Column(type: 'integer')]
    private int $user_id;
    
    #[Column(type: 'integer')]
    private int $post_id;
}
```

> Read more about [composite primary keys](/docs/en/advanced/composite-pk.md).

## Merging Attributes

The Annotated Entities extension supports merging table definitions from linked Mapper, Source, Repository, and Scope
classes. This approach is useful for implementing reusable domain functionality like timestamps or soft deletes.

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity(repository: Repository\UserRepository::class)]
class User
{
    // ...
}
```

Define schema extensions in your repository:

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Table\Index;
use Cycle\ORM\Select\Repository;

#[Column(name: 'created_at', type: 'datetime')]
#[Column(name: 'updated_at', type: 'datetime')]
#[Index(columns: ['created_at'])]
class UserRepository extends Repository
{
    // Custom repository methods
}
```

This pattern allows you to:

- Share common columns across multiple entities
- Implement cross-cutting concerns (auditing, soft deletes)
- Keep entity classes focused on business logic
- Reuse schema definitions through inheritance

> Read more about [custom repositories](/docs/en/basic/repository.md)
> and [entity behaviors](/docs/en/entity-behaviors/timestamps.md).

## Foreign Keys

In some cases, it is necessary to define a foreign key without relation definitions. This can be achieved using the
`#[ForeignKey]` attribute.

### Foreign Key at Property Level

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\ForeignKey;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;
}

#[Entity]
class Post
{
    #[Column(type: 'primary')]
    public int $id;
    
    #[Column(type: 'integer')]
    #[ForeignKey(target: User::class, action: 'CASCADE')]
    private int $userId;
}
```

### Foreign Key at Class Level

Alternatively, specify all foreign key details at the class level:

```php
#[Entity]
#[ForeignKey(
    target: User::class,
    innerKey: 'user_id',      // Column in this table
    outerKey: 'id',           // Column in target table
    action: 'CASCADE'
)]
class Post
{
    #[Column(type: 'primary')]
    public int $id;
    
    #[Column(type: 'integer')]
    private int $user_id;
}
```

### ForeignKey Attribute Specification

| Parameter   | Type          | Default   | Description                                                                                        |
|-------------|---------------|-----------|----------------------------------------------------------------------------------------------------|
| target      | string        | -         | **Required**. Target entity role or class name                                                     |
| innerKey    | string\|array | null      | Column(s) in source entity. Required for class-level FK, inferred from property for property-level |
| outerKey    | string\|array | null      | Column(s) in target entity. Defaults to primary key of target                                      |
| action      | string        | 'CASCADE' | Foreign key action for both DELETE and UPDATE: 'CASCADE', 'NO ACTION', 'SET NULL'                  |
| indexCreate | bool          | true      | Automatically create index on innerKey                                                             |

> **Note:** MySQL and SQL Server may automatically create indexes for foreign keys. The `indexCreate` option ensures
> cross-database compatibility.

> Read more about foreign keys in
> relations: [BelongsTo Foreign Keys](/docs/en/relation/belongs-to.md#foreign-key-options)
