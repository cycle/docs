# Configuring Schema Builder

Though the usage of annotated entities provides a very easy way to configure your ORM and Database schema, you might
want to use some alternative schema declaration notation, or perform some calculations on the ORM schema (for example
default values, custom columns and etc). This functionality is available in `cycle/schema-builder` extension.

## Manually Define Entity

We can define an ORM entity manually using the schema builder package. In order to do that we have to create `Registry`
object first:

```php
$registry = new \Cycle\Schema\Registry($dbal);
```

We can now register our first entity, add its columns and link to a specific table:

```php
use Cycle\Schema\Definition;

$entity = new Definition\Entity();

$entity
    ->setRole('user')
    ->setClass(User::class);

// add fields
$entity->getFields()
    ->set('id', (new Definition\Field())->setType('primary')->setColumn('id')->setPrimary(true))
    ->set('name', (new Definition\Field())->setType('string(32)')->setColumn('user_name'));

// register entity
$r->register($entity);

// associate table
$r->linkTable($entity, 'default', 'users');
```

You can generate ORM schema immediately using `Cycle\Schema\Compiler`:

```php
$schema = (new \Cycle\Schema\Compiler())->compile($r);

$orm = $orm->with(schema: new \Cycle\ORM\Schema($schema));
```

> You can also declare relations, indexes and associate custom mappers. See [examples](https://github.com/cycle/schema-builder/tree/master/tests/Schema).

## Custom Generators

Upon generating the final schema, you can pass a set of generators to your registry, each of them is responsible for the
specific part of the schema compilation. Simple compilation pipeline will look like:

```php
use Cycle\Schema;

$schema = (new Schema\Compiler())->compile($r, [
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

You can extend this pipeline by implementing your own generator:

```php
namespace Cycle\Schema;

interface GeneratorInterface
{
    /**
     * Run generator over given registry.
     */
    public function run(Registry $registry): Registry;
}
```

## Predefined schema properties

Upon generating the final schema, you can pass a set of default schema properties:

```php
use Cycle\Schema;
use Cycle\ORM\SchemaInterface;

$schema = (new Schema\Compiler())->compile($r, [/* ... */], [
    SchemaInterface::MAPPER => CustomMapper::class,
    SchemaInterface::REPOSITORY => CustomRepository::class,
    SchemaInterface::SOURCE => CustomSource::class,
    SchemaInterface::SCOPE => null,
    SchemaInterface::TYPECAST_HANDLER => [
        \Cycle\ORM\Parser\Typecast::class,
        CustomJsonTypecastHandler::class,
    ],
]);
```
