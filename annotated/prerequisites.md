# Prerequisites
Make sure to install `cycle/annoated` and `cycle/schema-builder` extenstions in order to use annotated entities. Once installed add 
annotated generators into schema compiler (see more details [here](/basic/install.md)).

## Compiler Pipeline
The complete pipeline with annotated entities support will look like:

```php
use Cycle\Schema;
use Cycle\Annotated;
use Spiral\Tokenizer;

// Class locator
$cl = (new Tokenizer\Tokenizer(new Tokenizer\Config\TokenizerConfig([
    'directories' => ['src/'],
])))->classLocator();

$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Embeddings($cl),            // register annotated embeddings
    new Annotated\Entities($cl),              // register annotated entities
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Annotated\Columns(),                  // register non field columns (table level)
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Annotated\Indexes(),                  // register non entity indexes (table level)
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);

$orm = $orm->withSchema(new Schema($schema));
```

> Make sure to point class locator the directory with your domain entities only as the indexation operation is fairly expensive. Make sure that all of the entities are loadable by `composer autoload`.

## Notes
Annotations are made Doctrine-like as a well established pattern. In your annotations you can use lower-cased or upper-cased notations. Both examples are valid:

```php
/**
 * @entity 
 */
```

```php
/**
 * @Entity 
 */
```
