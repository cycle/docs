# UUID as primary key
It is possible to use any type of primary sequence for your entities by altering the default `Cycle\ORM\Mapper\Mapper` behavior. In order to
do that we have to define a custom column with the `primary` flag.

> You can also specify a primary key value directly in your entity, the Mapper will accept it as the expected value.

## Entity
To define a specific column as the primary key, use the annotation option `primary`:

```php
/**
 * @Entity
 */
class User
{
    /** @Column(type = "string(36)", primary = true) */
    protected $uuid;

    public function getUUID(): string
    {
        return $uuid;
    }
}
```

## Mapper
In order to alter the default sequence behavior, alter the default mapper implementation by modifying the method `nextPrimaryKey`:

```php
use Ramsey\Uuid\Uuid;

class UUIDMapper extends Mapper
{
    /**
     * Generate entity primary key value.
     */
    public function nextPrimaryKey()
    {
        try {
            return Uuid::uuid4()->toString();
        } catch (\Exception $e) {
            throw new MapperException($e->getMessage(), $e->getCode(), $e);
        }
    }
}
```

## Link Entity and Mapper
In order to associate the mapper and the entity use the `@Entity` option `mapper`:

```php
/**
 * @Entity(mapper="Mapper\UUIDMapper")
 */
class User
{
    /** @Column(type = "string(36)", primary = true) */
    protected $uuid;
}
```

Now the entity's primary key will be generated on application end during the persist operation:

```php
$u = new User();

$t = new Transaction($orm);
$t->persist($u);
$t->run();

print_r($u->getUUID());
```
