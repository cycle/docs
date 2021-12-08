# Upgrade Guide

This guide will help you to upgrade CycleORM To 2.x From 1.x.

> We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of
> the ORM only a portion of these changes may actually affect your application.

## PHP >= 8.0 Required

> Likelihood Of Impact: High

The new minimum PHP version is now 8.0

## Updating Dependencies

> Likelihood Of Impact: High

Update the following dependencies in your composer.json file:

- `cycle/orm` to `^2.0`
- `spiral/database` replaced with `cycle/database`
- `cycle/database` to `^2.0`
- `cycle/proxy-factory` is no longer needed
- `cycle/annotated` to `^2.0`
- `cycle/schema-builder` to `^2.0`
- `cycle/migrations:^1.0` replaced with `cycle/schema-migrations-generator:^1.0`
- `spiral/migrations` replaced with `cycle/migrations:^2.0`

## Namespaces

> Likelihood Of Impact: High

`spiral/database` is moved to a new repository `cycle/database` so now it is namespaced. To accommodate for these
changes you need to replace all namespaces start from `Spiral\Database` with `Cycle\Database`

## Proxy factory

> Likelihood Of Impact: High

Since CycleORM v2.0 you don't need `cycle/proxy-factory` package anymore. ORM uses proxy entities out of the box.

## Transaction

> Likelihood Of Impact: Low

CycleORM deleting and persisting features have been rewritten to support retries, deferred persists and
states. `Cycle\ORM\Transaction` class was marked as deprecated. Use `Cycle\ORM\EntityManager`
instead. [Read more](/docs/en/advanced/entity-manager.md)

## Collections

> Likelihood Of Impact: Medium

CycleORM doesn't use `doctrine/collections` out of the box anymore. If you want to continue using it you need to add
package `doctrine/collections` to `composer.json` and use `Cycle\ORM\Collection\DoctrineCollectionFactory`
as `defaultCollectionFactory` in your `Cycle\ORM\Factory` object.

```php
use Cycle\ORM;

$schema = new ORM\Schema(...);

$factory = (new ORM\Factory(
    dbal: $dbal,
    defaultCollectionFactory: new ORM\Collection\DoctrineCollectionFactory
))
```

## Entity annotations

> Likelihood Of Impact: Optional

Since `cycle/annotated` v2.0 we have added better support for PHP8 attributes and you have an ability to use the instead
of annotations.

## ORM

> Likelihood Of Impact: High

`Cycle\ORM\ORMInterface ` interface has been updated. There are new methods and changes in exist methods.

#### ORM with methods

Methods `withFactory(FactoryInterface $factory)`, `withSchema(SchemaInterface $schema)`
and `withHeap(HeapInterface $heap)` marked as deprecated, use method `with` instead:

```php
$orm = new \Cycle\ORM\ORM(...);

$orm->with(
    schema: new \Cycle\ORM\Schema(...),
    factory: new \Cycle\ORM\Factory(...),
    heap: new \Cycle\ORM\Heap\Heap(...)
);
```

## Mapper

> Likelihood Of Impact: High

`Cycle\ORM\MapperInterface` interface has been updated. There are new methods and changes in exist methods.

## Factory

> Likelihood Of Impact: High

`Cycle\ORM\FactoryInterface` interface has been updated. There are new methods and changes in exist methods.

## Entity hydrator

> Likelihood Of Impact: Medium

CycleORM doesn't use `laminas/laminas-hydrator` anymore. The ORM has its own
`Cycle\ORM\Mapper\Proxy\Hydrator\ClosureHydrator` underhood, it works faster and supports private entity properties.
