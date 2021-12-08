# Column Wrappers
In some cases, you might want to wrap the value using a custom value object (similarly to DateTime columns which are wrapped as DateTimeImmutable).
It can be achieved by creating a custom column wrapper and typecasting the column value to it.

> Note, all column wrappers must be immutable, you have to reassign property value to trigger the change.

## Example
In order to define a column wrapper, we have to implement an object with a static `typecast` method. We will use a UUID column as an example.

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Cycle\Database\DatabaseInterface;

class Uuid
{
    /** @var UuidBody */
    private $uuid;

    /**
     * @return string
     */
    public function __toString()
    {
        return $this->uuid->toString();
    }

    /**
     * @return Uuid
     * @throws \Exception
     */
    public static function create(): Uuid
    {
        $uuid = new static();
        $uuid->uuid = UuidBody::uuid4();
        return $uuid;
    }

    /**
     * @param string            $value
     * @param DatabaseInterface $db
     * @return Uuid
     */
    public static function typecast($value, DatabaseInterface $db): Uuid
    {
        $uuid = new static();
        $uuid->uuid = UuidBody::fromString((string)$value);
        return $uuid;
    }
}
```

> Please note that the `typecast` method will receive the raw value content and the database it's associated with. Make sure to implement the `__toString`
method on your wrapper to store it in the database. See below how to use a custom serialization strategy.

## Assign to entity
To assign a column wrapper to an entity use the column option `typecast`. You can specify typecast as a function name, a method name (:: separator), or
a class name which defines static method typecast:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public $id;

    #[Column(type: 'string', typecast: Uuid::class)]
    public $uuid;
}
```

We can use this column wrapper after the schema update:

```php
$u = new User();
$u->uuid = Uuid::create();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u);
$state = $manager->run();
```

The column will be automatically wrapped upon retrieving the entity from the database:

```php
$u = $orm->getRepository(User::class)->findOne();
print_r($u);
```

To change the wrapped column value you have to create new object:

```php
$u = $orm->getRepository(User::class)->findOne();
$u->uuid = Uuid::create();

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u);
$state = $manager->run();
```

## Raw Values
In some cases you might want to store values in the database in binary form, you can achieve that by implementing `Cycle\Database\Injection\ValueInterface`
in order to gain access to low-level query compilation:

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Cycle\Database\DatabaseInterface;
use Cycle\Database\Injection\ValueInterface;

class Uuid implements ValueInterface
{
    private UuidBody $uuid;

    public function rawValue(): string
    {
        return $this->uuid->getBytes();
    }

    public function rawType(): int
    {
        return \PDO::PARAM_LOB;
    }

    public function __toString()
    {
        return $this->uuid->toString();
    }

    public static function create(): Uuid
    {
        $uuid = new static();
        $uuid->uuid = UuidBody::uuid4();
        return $uuid;
    }

    public static function typecast(string $value, DatabaseInterface $db): static
    {
        if (is_resource($value)) {
            // postgres
            $value = fread($value, 16);
        }

        $uuid = new static();
        $uuid->uuid = UuidBody::fromBytes((string)$value);
        return $uuid;
    }
}
```

Now, the Uuid column will be stored in a blob form.

> You can also implement the Expression or Parameter interfaces in your column wrapper to achieve more complex logic.
