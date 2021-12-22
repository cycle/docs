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
#[Uuid1(field: 'uuid', node: '00000fffffff', clockSeq: 0xffff)]
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
    clockSeq: 0xffff
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
#[Uuid3(
    field: 'uuid',
    namespace: Uuid::NAMESPACE_URL,
    name: 'https://example.com/foo'
)]
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
#[Uuid4]
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
#[Uuid5(
    field: 'uuid', 
    namespace: Uuid::NAMESPACE_URL, 
    name: 'https://example.com/foo'
)]
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
#[Uuid6(
    field: 'uuid', 
    node: '00000fffffff', 
    clockSeq: 0xffff
)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

> Note: If you have a custom `uuid` column declaration, it should be compatible ith `Behavior\Uuid\Uuid*` column type, 
> otherwise an exception will be thrown.
