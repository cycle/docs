# Entity behaviors

The `cycle/entity-behavior` package is a collection of attributes that add behaviors to Cycle ORM entities. The package
also provides a simple API to create custom behavior attributes.

The package provides extended command generator (`Cycle\ORM\Entity\Behavior\EventDrivenCommandGenerator`), that will
fire events on create, update and delete entity. The generator dispatch several events, allowing you to hook into the
following moments in a entity's lifecycle: creating, created, updating, updated,deleting and deleted.

## Installation

The package is available via composer and can be installed using the following command:

```bash
composer require cycle/entity-behavior
```

## Configuration

After installation the package you need to create `Cycle\ORM\ORM` object with
passing `\Cycle\ORM\Entity\Behavior\EventDrivenCommandGenerator` generator object as third (`commandGenerator`)
argument.

**Example**

```php
use Cycle\ORM\ORM;
use Cycle\ORM\Entity\Behavior\EventDrivenCommandGenerator;

// Application container (PSR-11 compatible).
// https://www.php-fig.org/psr/psr-11/
$container = new Container();
$commandGenerator = new EventDrivenCommandGenerator($schema, $container);

$orm = new ORM(
  factory: $factory, 
  schema: $schema, 
  commandGenerator: $commandGenerator
);
```

That's it. Now you can use all benefits of this package.

### Available behaviors

- [UUID](/docs/en/entity-behaviors/uuid.md)
- [CreatedAt и UpdatedAt](/docs/en/entity-behaviors/timestamps.md)
- [SoftDelete](/docs/en/entity-behaviors/soft-delete.md)
- [OptimisticLock](/docs/en/entity-behaviors/optimistic-lock.md)
- [Hook](/docs/en/entity-behaviors/hooks.md)
- [EventListener](/docs/en/entity-behaviors/event-listener.md)

