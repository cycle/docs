# OptimisticLock

The package `cycle/entity-behavior` adds support for automatic optimistic locking via a version field. In this approach
any entity that should be protected against concurrent modifications during long-running business transactions gets a
version field. When changes to such an entity are persisted at the end of a long-running conversation the version of the
entity is compared to the version in the database and if they don't match,
an `Cycle\ORM\Entity\Behavior\Exception\OptimisticLock\RecordIsLockedException` is thrown, indicating that the entity
has been modified by someone else already.

## Available strategies

- `MICROTIME` - current timestamp with microseconds as string
- `RAND_STR` - Random generated string (random_bytes)
- `INCREMENT` - Auto incremented integer version
- `DATETIME` - Current datetime
- `MANUAL` - Allows using custom realisation for setting and controlling version. You have to manage entity and DB
  column by yourself.

## Usage:

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\OptimisticLock;

#[Entity]
#[OptimisticLock(
  field: 'version',                         // Required. By default 'version' 
  column: 'version',                        // Optional. By default 'null'. If not set, will be used information from property declaration.
  rule: OptimisticLock::RULE_INCREMENT      // Optional. By default OptimisticLock::RULE_INCREMENT
)]
class Page
{
    #[Column(type: 'primary')]
    private int $id;
    
    #[Column(type: 'integer')]
    private int $version;
}
```

UUID behavior attribute has the ability to manage entity column, so it's unnecessary to use `#[Column]` attribute.

> Note: If you have a custom `field` column declaration, it should be compatible with `Behavior\OptimisticLock` column 
> type, otherwise an exception will be thrown.
