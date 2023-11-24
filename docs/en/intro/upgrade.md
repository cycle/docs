# Upgrade Guide

This guide will help you to upgrade Cycle ORM to 2.x from 1.x.

> We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of
> the ORM only a portion of these changes may actually affect your application.

## PHP >= 8.0 required

> Likelihood of impact: High

The new minimum PHP version is now 8.0

## Updating dependencies

> Likelihood of impact: High

Update the following dependencies in your composer.json file:

- `cycle/orm` to `^2.0`
- `spiral/database` replace with `cycle/database`
- `cycle/database` to `^2.0`
- `cycle/proxy-factory` is no longer needed
- `cycle/annotated` to `^3.0`
- `cycle/schema-builder` to `^2.0`
- `cycle/migrations:^1.0` replace with `cycle/schema-migrations-generator:^2.0`
- `spiral/migrations` replace with `cycle/migrations:^3.0`

## Namespaces

> Likelihood of impact: High

`spiral/database` is moved to a new repository `cycle/database` so now it has new namespace. To accommodate for these
changes you need to replace all namespaces start from `Spiral\Database` with `Cycle\Database`

## Database config

> Likelihood of impact: High

Since `cycle/database` v2.0 connection configuration has been changed. You don't need to configure arrays anymore. Use
cofing DTO's instead of. Read more on [database connection](/docs/en/database/connect.md) page.

## Database logger

> Likelihood of impact: Medium

`Cycle\Database\DatabaseManager` doesn't use `spiral/core` package anymore. If you need to use driver specific logger,
you have to create your own LoggerFactory implementing `Cycle\Database\LoggerFactoryInterface`.
Read more on [database profiling](/docs/en/database/profiling.md) page.

## Proxy factory

> Likelihood of impact: High

Since Cycle ORM v2.0 you don't need `cycle/proxy-factory` package anymore. ORM uses proxy entities out of the box.

## Transaction

> Likelihood of impact: Low

`Cycle\ORM\Transaction` class was marked as deprecated. Use `Cycle\ORM\EntityManager`
instead. [Read more](/docs/en/advanced/entity-manager.md)

## Collections

> Likelihood of impact: Medium

Cycle ORM doesn't use `doctrine/collections` out of the box anymore. If you want to continue using it you need to add
package `doctrine/collections` to `composer.json` and use `Cycle\ORM\Collection\DoctrineCollectionFactory`
as a `defaultCollectionFactory` in your `Cycle\ORM\Factory` object.

```php
use Cycle\ORM;

$factory = (new ORM\Factory(
    dbal: $dbal,
    defaultCollectionFactory: new ORM\Collection\DoctrineCollectionFactory
));
```

## ORM

> Likelihood of impact: High

The second argument of the `Cycle\ORM\ORM` constructor is now required.

## Mapper

> Likelihood of impact: Medium

Default mapper `Cycle\ORM\Mapper\Mapper` is completely reworked. Now it works as a Proxy mapper and has its own hydrator
instead of `laminas/laminas-hydrator` - `Cycle\ORM\Mapper\Proxy\Hydrator\ClosureHydrator` and it works faster and
supports private and typed entity properties.

Pay attention that `Cycle\ORM\MapperInterface` has BC changes, and you need to rework your custom mappers.
`queueDelete()`, `queueUpdate()` and `queueCreate()` methods also has been changed.

> Custom mappers like `SoftDeleteMapper`, `OptimisticLockMapper`, `UuidMapper` in the ORM v2.0 can be implemented
> via [behaviors](/docs/en/entity-behaviors/install.md)

> Some typical custom mapper use cases like `SoftDelete`, `OptimisticLock`
> available in the [behaviors](/docs/en/entity-behaviors/install.md).

## Constraint

> Likelihood of impact: High

Everything associated with Constrain in the ORM is replaced with Scope:

 - the `Cycle\ORM\SchemaInterface::CONSTRAIN` constant, marked deprecated in ORM v1, is removed;
 - Loader option `'constrain'` -> `'scope'`
 - `Cycle\ORM\Select::setConstrain()` -> `Cycle\ORM\Select::setScope()`
 - `Cycle\ORM\Select\ConstrainInterface` -> `Cycle\ORM\Select\ScopeInterface`

## ORM `with...` methods

> Likelihood of impact: Medium

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

## Manual Iterator creation

> Likelihood of impact: Medium

Since Cycle ORM v2.0, need to create a `Cycle\ORM\Iterator` object using the static methods `createWithOrm`
or `createWithServices`:

```php
$iterator =  \Cycle\ORM\Iterator::createWithOrm($orm, 'user', $data);
```

## Entity annotations

> Likelihood of impact: Optional

Since `cycle/annotated` v2.0 we have added better support for PHP8 attributes, and you have an ability to use attributes
instead of annotations.

## Schema builder

> Likelihood of impact: Optional

Since `cycle/schema-builder` v2.0 we have added a new generators for STI/JTI and schema modifiers support. If you want
to use new features you have to add them to the schema compiler pipeline.

```php
[
    new Schema\Generator\ResetTables(),
    new Annotated\Embeddings($classLocator),
    new Annotated\Entities($classLocator),
    new Annotated\TableInheritance(),               // <------ register STI/JTI
    new Annotated\MergeColumns(),
    new Schema\Generator\GenerateRelations(),
    new Schema\Generator\GenerateModifiers(),       // <----- generate changes from schema modifiers
    new Schema\Generator\ValidateEntities(),
    new Schema\Generator\RenderTables(),
    new Schema\Generator\RenderRelations()
    new Schema\Generator\RenderModifiers(),         // <----- render all schema modifiers
    new Schema\Generator\ForeignKeys(),             // Since cycle/schema-builder v2.6.0. Define foreign key constraints
    new Annotated\MergeIndexes(),
    new Schema\Generator\SyncTables(),              // Not for production
    new Schema\Generator\GenerateTypecast(),
]
```
