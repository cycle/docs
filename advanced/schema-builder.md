# Configuring Schema Builder
Though using annotated entities provide very easy way to configure your ORM and Database schema you might want to use some alternative
schema declaration notation or perform some calculations on ORM schema (for example default values, custom columns and etc). This functionality is available in `cycle/schema-builder` extension.

## Manually Define Entity
We can define ORM entity manually using schema builder package, in order to do that we have to create `Registry` object first:

```php
$r = new Schema\Registry($dbal);
```

We can now register our first entity, add it columns and link to specific table:

```php
use Cycle\Schema\Definition;

$e = new Definition\Entity();
$e->setRole('user');
$e->setClass(User::class);

// add fields
$entity->getFields()->set(
    'id', (new Field())->setType('primary')->setColumn('id')->setPrimary(true)
);

$entity->getFields()->set(
    'name',
    (new Field())->setType('string(32)')->setColumn('user_name')
);

// register entity
$r->register($e);

// associate table
$r->linkTable($e, 'default', 'users');
```

You can generate ORM schema immediatelly using `Schema\Compiler`:

```php
$schema = (new Schema\Compiler())->compile($r, []);

$orm = $orm->withSchema(new ORM\Schema($schema));
```

> You can also declare relations, indexes and associate custom mappers. See [examples](https://github.com/cycle/schema-builder/tree/master/tests/Schema).

## Custom Generators
Upon generating final schema you must pass your registry should set of generators, each of them is response for specific part of
schema compilation. Simple compilation pipeline will look like:

```php
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
     *
     * @param Registry $registry
     * @return Registry
     */
    public function run(Registry $registry): Registry;
}
```
