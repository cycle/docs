# Identifiers

The `cycle/entity-behavior-identifier` package provides the ability to use `ramsey/identifier` as various Cycle ORM entity column types.

## Installation

> **Note:** Due to a dependency on `ramsey/identifier` this package requires PHP `8.2` or newer.

Install this package as a dependency using Composer.

```bash
composer require cycle/entity-behavior-identifier
```

## Usage

The package provides various types of identifiers, for generating unique values or as alternatives to auto-increment
IDs, helping ensure uniqueness and flexibility across an application.

> **Note: ** Most identifiers encode metadata such as node ID and epoch offset. These values are typically
> derived from the platform or system rather than defined within an entity. Each applicable listener class provides a
> `setDefaults` method to allow these values to be set at an appropriate time within your application.

For example:

```php
\Cycle\ORM\Entity\Behavior\Identifier\Listener\SnowflakeGeneric::setDefaults(0, 1_446_940_800_000);
\Cycle\ORM\Entity\Behavior\Identifier\Listener\Uuid1::setDefaults('00000fffffff', 0xffff);
```


### Snowflake Examples

**Snowflake (Generic):** A flexible Snowflake implementation that generates globally unique, time-ordered 64-bit IDs
without adhering to any specific platform’s conventions, suitable for general distributed systems.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Snowflake;

#[Entity]
#[Identifier\SnowflakeGeneric(field: 'id')]
class User
{
    #[Column(type: 'id', primary: true)]
    public Snowflake $id;
}
```

**Snowflake (Discord):** Implements Discord’s Snowflake format, generating 64-bit IDs that encode a timestamp,
worker ID, and sequence number. Useful when interoperating with Discord’s API or matching its ID structure.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Snowflake;

#[Entity]
#[Identifier\SnowflakeDiscord(field: 'id')]
class User
{
    #[Column(type: 'id', primary: true)]
    public Snowflake $id;
}
```

**Snowflake (Instagram):** Follows Instagram’s Snowflake structure to produce unique, sortable 64-bit IDs suitable for
applications that need compatibility with Instagram-style ID sequences.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Snowflake;

#[Entity]
#[Identifier\SnowflakeInstagram(field: 'id')]
class User
{
    #[Column(type: 'id', primary: true)]
    public Snowflake $id;
}
```

**Snowflake (Mastodon):** Generates IDs compatible with Mastodon’s distributed Snowflake system, encoding time and node
information to ensure uniqueness across instances.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Snowflake;

#[Entity]
#[Identifier\SnowflakeMastodon(field: 'id')]
class User
{
    #[Column(type: 'id', primary: true)]
    public Snowflake $id;
}
```

**Snowflake (Twitter):** Produces 64-bit IDs in the format used by Twitter, encoding timestamp, machine ID, and sequence
number for globally unique, time-sortable identifiers. Ideal for high-throughput distributed systems.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Snowflake;

#[Entity]
#[Identifier\SnowflakeTwitter(field: 'id')]
class User
{
    #[Column(type: 'id', primary: true)]
    public Snowflake $id;
}
```

### ULID Examples

**ULID (Universally Unique Lexicographically Sortable Identifier):** A 128-bit identifier designed for high uniqueness
and lexicographical sortability. It combines a timestamp component with random data, allowing for ordered IDs that can
be generated rapidly and are human-readable, making it ideal for databases and distributed systems.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Ulid;

#[Entity]
#[Identifier\Ulid(field: 'id')]
class User
{
    #[Column(type: 'ulid', primary: true)]
    private Ulid $id;
}
```

### UUID Examples

**UUID Version 1 (Time-based):** Generated using the current timestamp and the MAC address of the computer, ensuring
unique identification based on time and hardware.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid1(field: 'id')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 2 (DCE Security):** Similar to version 1 but includes a local identifier such as a user ID or group ID,
primarily used in DCE security contexts.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid2(field: 'id')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 3 (Name-based, MD5):** Created by hashing a namespace identifier and name using MD5, resulting in a
deterministic UUID based on input data.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid3(
    field: 'id',
    namespace: '6ba7b810-9dad-11d1-80b4-00c04fd430c8',
    name: 'example.com',
)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 4 (Random):** Generated entirely from random or pseudo-random numbers, offering high unpredictability
and uniqueness.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid4(field: 'id')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 5 (Name-based, SHA-1):** Similar to version 3 but uses SHA-1 hashing, providing a different deterministic
UUID based on namespace and name.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid5(
    field: 'id',
    namespace: '6ba7b810-9dad-11d1-80b4-00c04fd430c8',
    name: 'example.com',
)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 6 (Draft/Upcoming):** An experimental or proposed version focused on improving time-based UUIDs with more
sortable properties (not yet widely adopted).

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid6(field: 'id')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```

**UUID Version 7 (Draft/Upcoming):** A newer proposal designed to incorporate sortable features based on Unix timestamp,
enhancing performance in database indexing.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Identifier;
use Ramsey\Identifier\Uuid;

#[Entity]
#[Identifier\Uuid7(field: 'id')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private Uuid $id;
}
```
