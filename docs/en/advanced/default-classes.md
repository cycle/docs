# Customize Schema Defaults

You can set default [Repository](/docs/en/basic/repository.md), [Mapper](/docs/en/advanced/mapper.md), Source
or [Scope](/docs/en/advanced/scope.md) for all entity classes

```php
use Cycle\ORM\Factory;
use Cycle\ORM\ORM;
use Cycle\ORM\Schema;

$orm = new ORM(
    (new Factory($dbal))->withDefaultSchemaClasses([
        SchemaInterface::REPOSITORY => MyRepository::class,
        SchemaInterface::SOURCE => MySource::class,
        SchemaInterface::MAPPER => MyMapper::class,
        SchemaInterface::SCOPE => MyScope::class,
        SchemaInterface::TYPECAST_HANDLER => [
            \Cycle\ORM\Parser\Typecast::class,
            UuidTypecast::class,
        ],
    ])
);
```

You can change any of these values, if you want.

**By default, this config has the following values:**

```php
[
    SchemaInterface::REPOSITORY => \Cycle\ORM\Select\Repository::class,
    SchemaInterface::SOURCE => \Cycle\ORM\Select\Source::class,
    SchemaInterface::MAPPER => \Cycle\ORM\Mapper\Mapper::class,
    SchemaInterface::SCOPE => null,
    SchemaInterface::TYPECAST_HANDLER => null,
]
```
