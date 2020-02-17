# Customize Schema
You can set default Repository, Mapper, Source or Constrain for all entity classes

```php

$orm = new ORM((new Factory())->withDefaultSchemaClasses(
    [
        Schema::REPOSITORY => MyRepository::class,
        Schema::SOURCE     => MySource::class,
        Schema::MAPPER     => MyMapper::class,
        Schema::CONSTRAIN  => MyConstrain::class,
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
        Schema::CONSTRAIN  => null,
    ]

```
