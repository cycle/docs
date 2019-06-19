# Column Wrappers
In some cases you might want to wrap the value using custom value object (similary to datetime columns which are wrapped as DateTimeImmutable).
It can be achieved by creating custom column wrapper and typecasting column value to it. 

> Note, all column wrappers must be immutable, you have to reassing properly value to trigger the change.

## Example
In order to define column wrapper we have to implement object with static `typecast` method. We would use UUID column as example.

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Spiral\Database\DatabaseInterface;

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

> Please note that, `typecast` method will receive raw value content and database it's associated with. Make sure to implement `__toString`
method on your wrapper to store it in database.

## Assing to entity
To assign column wrapper to entity use column option `typecast`, you can specify typecast as function name, method name (:: separater) or 
class name which defines static method typecast:

```php
/** @entity */
class User
{
    /** @column(type=primary) */
    public $id;

    /** @column(type=string, typecast="Uuid") */
    public $uuid;
}
```

We can use this column wrapper after the schema update:

```php
$u = new User();
$u->uuid = Uuid::create();

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

Column will be automatically wrapped upon selection entity from the database:

```php
$u = $orm->getRepository(User::class)->findOne();
print_r($u);
```

To change wrapped column value you have to create new object:

```php
$u = $orm->getRepository(User::class)->findOne();
$u->uuid = Uuid::create();

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

## Raw Values
In some cases you might want to store values in database in binary form, you can achieve that by implementing `Spiral\Database\Injection\ValueInterface`
in order to gain access to low level query compilation:

```php
use Ramsey\Uuid\Uuid as UuidBody;
use Spiral\Database\DatabaseInterface;
use Spiral\Database\Injection\ValueInterface;

class Uuid implements ValueInterface
{
    /** @var UuidBody */
    private $uuid;

    /**
     * @return string
     */
    public function rawValue(): string
    {
        return $this->uuid->getBytes();
    }

    /**
     * @return int
     */
    public function rawType(): int
    {
        return \PDO::PARAM_LOB;
    }

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

Now, Uuid column will be stored in a blob form.

> You can also implement Expression or Parameter interfaces in your column wrapper to achieve more complex logic.
