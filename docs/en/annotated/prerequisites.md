# Prerequisites

Make sure to install `cycle/annotated` and `cycle/schema-builder` extensions in order to use annotated entities. Once
installed add annotated generators into the schema compiler (see more details [here](/docs/en/intro/install.md)).

> Cycle is using the Doctrine/Annotations package, make sure that annotations are loadable (`use`) and the syntax 
> is correct.

## Compiler Pipeline

The complete pipeline with annotated entities support will look like:

```php
use Cycle\Schema;
use Cycle\Annotated;
use Spiral\Tokenizer;

// Class locator
$classLocator = (new Tokenizer\Tokenizer(new Tokenizer\Config\TokenizerConfig([
    'directories' => ['src/'],
])))->classLocator();

$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Schema\Generator\ResetTables(),             // re-declared table schemas (remove columns)
    new Annotated\Embeddings($classLocator),        // register embeddable entities
    new Annotated\Entities($classLocator),          // register annotated entities
    new Annotated\TableInheritance(),               // register STI/JTI
    new Annotated\MergeColumns(),                   // add @Table column declarations
    new Schema\Generator\GenerateRelations(),       // generate entity relations
    new Schema\Generator\GenerateModifiers(),       // generate changes from schema modifiers
    new Schema\Generator\ValidateEntities(),        // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),            // declare table schemas
    new Schema\Generator\RenderRelations(),         // declare relation keys and indexes
    new Schema\Generator\RenderModifiers(),         // render all schema modifiers
    new Annotated\MergeIndexes(),                   // add @Table column declarations
    new Schema\Generator\SyncTables(),              // sync table changes to database
    new Schema\Generator\GenerateTypecast(),        // typecast non string columns
]);

$orm = $orm->with(schema: new \Cycle\ORM\Schema($schema));
```

> Make sure to point the class locator to the directory with your domain entities only as the indexation operation 
> is fairly expensive. Make sure that all of the entities are loadable by `composer autoload`.

The result of the schema builder is a compiled schema. The given schema can be cached in order to avoid expensive
calculations on each request.

> In the following section the terming `update schema` will be referenced to this process.

Remove element `new Schema\Generator\SyncTables()` to disable database reflection. In a later section, we will describe
how to automatically render database migrations instead of direct synchronization.
