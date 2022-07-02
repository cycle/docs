# Column values typecast

In some cases, you might want to wrap the value using a custom value object (similarly to DateTime columns which are
wrapped as `DateTimeImmutable`). It can be achieved by creating a custom column wrapper and typecasting the column value
to it.

## Typecast using callable

In order to define a column wrapper, we have to implement an object with a static factory method. 
We will use the UUID column and the `castValue()` method as an example.

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Ramsey\Uuid\UuidInterface;
use Cycle\Database\DatabaseInterface;

final class Uuid
{
    private function __construct(
        private UuidInterface $uuid;
    ) {
    }

    public function __toString(): string
    {
        return $this->uuid->toString();
    }

    public static function create(): static
    {
        return new static(
            UuidBody::uuid4()
        );
    }

    public static function castValue(string $value, DatabaseInterface $db): static
    {
        return new static(
            UuidBody::fromString($value)
        );
    }
}
```

> **Note**
> The `castValue` method will receive the raw value content and the database it's associated with.
> Make sure to implement the `__toString` method on your wrapper to store it in the database.
> See below how to use a custom serialization strategy.

### Assign to entity

To assign a column wrapper to an entity use the column option `typecast`. You can specify `typecast` as any callable:
function name, a method name (using `::` separator or declaring as array).

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    private $id;

    #[Column(type: 'string', typecast: [Uuid::class, 'castValue'])]
    private Uuid $uuid;
}
```

> **Note**
> In the example we've declared a typecast rule to the `uuid` field. By default ORM will process the rule using default
> [Typecast Handlers](./typecasting.md) that allows using for callables.

We can use this column wrapper after the schema update:

```php
$user = new User();
$user->uuid = Uuid::create();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

The column will be automatically wrapped upon retrieving the entity from the database:

```php
$user = $orm->getRepository(User::class)->findOne();

print_r($user);
```

To change the wrapped column value you have to create new object:

```php
$user = $orm->getRepository(User::class)->findOne();
$user->uuid = Uuid::create();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

## Uncast Raw Values

In some cases you might want to store values in the database in binary form, you can achieve that by
implementing `Cycle\Database\Injection\ValueInterface` in order to gain access to low-level query compilation:

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Ramsey\Uuid\UuidInterface;
use Cycle\Database\DatabaseInterface;
use Cycle\Database\Injection\ValueInterface;

class Uuid implements ValueInterface
{
    private function __construct(
        private UuidInterface $uuid;
    ) {
    }
    
    public function rawValue(): string
    {
        return $this->uuid->getBytes();
    }

    public function rawType(): int
    {
        return \PDO::PARAM_LOB;
    }

    public function __toString(): string
    {
        return $this->uuid->toString();
    }

    public static function create(): static
    {
        return new static(
            UuidBody::uuid4()
        );
    }

    public static function castValue(string $value, DatabaseInterface $db): static
    {
        if (is_resource($value)) {
            // postgres
            $value = fread($value, 16);
        }

        return new static(
            UuidBody::fromBytes($value)
        );
    }
}
```

Now, the Uuid column will be stored in a blob form.

> You can also implement the Expression or Parameter interfaces in your column wrapper to achieve more complex logic.
