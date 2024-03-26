# UUID

The `cycle/entity-behavior-uuid` package provides the ability to use `ramsey/uuid` as a Cycle ORM property type.

## Installation

Install this package as a dependency using Composer.

```bash
composer require cycle/entity-behavior-uuid
```

## Usage

The package provides several version of UUID you can use for entities.

### RFC 4122 UUIDs

#### Version 1: Time-based

A version 1 UUID uses the current time, along with the MAC address (or node) for a network interface on the local
machine.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid1;
use Ramsey\Uuid\UuidInterface;

#[Entity]
#[Uuid1(field: 'uuid', node: '00000fffffff', clockSeq: 0xffff, nullable: false)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 2: DCE Security

UUID v2 uses the current time, along with the MAC address (or node) for a network interface on the local machine.
Additionally, a version 2 UUID replaces the low part of the time field with a local identifier such as the user ID or
group ID of the local account that created the UUID.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid2;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid2(
    field: 'uuid',
    localDomain: Uuid::DCE_DOMAIN_PERSON,
    localIdentifier: '12345678',
    node: '00000fffffff',
    clockSeq: 0xffff,
    nullable: false
)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 3: Name-based (MD5)

Uses a version 3 (name-based) UUID based on the MD5 hash of a namespace ID and a name

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid3;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid3(field: 'uuid', namespace: Uuid::NAMESPACE_URL, name: 'https://example.com/foo', nullable: false)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 4: Random

They are randomly-generated and do not contain any information about the time they are created or the machine that
generated them.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid4;
use Ramsey\Uuid\UuidInterface;

#[Entity]
#[Uuid4(nullable: false)]
class User
{
    #[Column(field: 'uuid', type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 5: Name-based (SHA-1)

Uses a version 5 (name-based) UUID based on the SHA-1 hash of a namespace ID and a name

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid5;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid5(field: 'uuid', namespace: Uuid::NAMESPACE_URL, name: 'https://example.com/foo', nullable: false)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 6: Nonstandard UUIDs

Ordered-Time Uses a version 6 (ordered-time) UUID from a host ID, sequence number, and the current time

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid6;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid6(field: 'uuid', node: '00000fffffff', clockSeq: 0xffff, nullable: false)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 7: Unix Epoch Time

Version 7 UUIDs solve two problems that have long existed with the use of version 1 UUIDs:
 - Scattered database records
 - Inability to sort by an identifier in a meaningful way (i.e., insert order)
To overcome these issues, we need the ability to generate UUIDs that are monotonically increasing.
Version 6 UUIDs provide an excellent solution for those who need monotonically increasing, sortable UUIDs with
the features of version 1 UUIDs (MAC address and clock sequence), but if those features aren’t necessary for your
application, using a version 6 UUID might be overkill. Version 7 UUIDs combine random data (like version 4 UUIDs) with
a timestamp (in milliseconds since the Unix Epoch, i.e., 1970-01-01 00:00:00 UTC) to create a monotonically increasing,
sortable UUID that doesn’t have any privacy concerns, since it doesn’t include a MAC address.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid7;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid7(field: 'uuid', nullable: false)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Disabling UUID autogeneration

In all UUID attributes, there is a `nullable` parameter. If you set this parameter to **true**, it will disable automatic
UUID generation. In this case, your database field must be nullable, or you must generate a value for the field yourself.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Uuid\Uuid4;
use Ramsey\Uuid\UuidInterface;

#[Entity]
#[Uuid4(field: 'token', nullable: true)]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'uuid', nullable: true)]
    private ?UuidInterface $token = null
}
```

> **Warning**
> If you have a custom `uuid` column declaration, it should be compatible with `Behavior\Uuid\Uuid*` column type,
> otherwise an exception will be thrown.
