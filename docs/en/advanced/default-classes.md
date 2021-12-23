# Customize Schema Defaults
You can set default Repository, Mapper, Source or Scope for all entity classes

```php
use Cycle\ORM\Factory;
use Cycle\ORM\ORM;
use Cycle\ORM\Schema;

$orm = new ORM((new Factory($dbal))->withDefaultSchemaClasses(
    [
        Schema::REPOSITORY => MyRepository::class,
        Schema::SOURCE     => MySource::class,
        Schema::MAPPER     => MyMapper::class,
        Schema::SCOPE      => MyScope::class,
    ]
));

```

You can change any of this values, if you want.

By default this config has this values:

```php
    [
        Schema::REPOSITORY => \Cycle\ORM\Select\Repository::class,
        Schema::SOURCE     => \Cycle\ORM\Select\Source::class,
        Schema::MAPPER     => \Cycle\ORM\Mapper\Mapper::class,
        Schema::SCOPE      => null,
    ]

```
