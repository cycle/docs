# UUID as primary key
It is possible to use any type of primary sequence for your entities by altering default `Cycle\ORM\Mapper\Mapper` behaviour. In order to
do that we have to define custom column with `primary` flag.

> You can also specify primary key directly in your entity, Mapper will accept it as expected value.

## Entity
To define specific column as primary key use annotation option `primary`:

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
In order to alter the default sequence behaviour alter the default mapper implementation by modifying method `nextPrimaryKey`:

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
In order to associate mapper and entity use `@Entity` option mapper:

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

Now, entity primary key will be generated on application end during the persist operation:

```php
$u = new User();

$t = new Transaction($orm);
$t->persist($u);
$t->run();

print_r($u->getUUID());
```
