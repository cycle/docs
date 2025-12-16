# Prerequisites

Make sure to install `cycle/annotated` and `cycle/schema-builder` extensions in order to use annotated entities:

```bash
composer require cycle/annotated cycle/schema-builder
```

Once installed, add annotated generators into the schema compiler (see more details
in [Installation Guide](/docs/en/intro/install.md)).

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
    new Schema\Generator\ResetTables(),             // Re-declare table schemas (remove columns)
    new Annotated\Embeddings($classLocator),        // Register embeddable entities
    new Annotated\Entities($classLocator),          // Register annotated entities
    new Annotated\TableInheritance(),               // Register STI/JTI
    new Annotated\MergeColumns(),                   // Register columns from attributes
    new Schema\Generator\GenerateRelations(),       // Generate entity relations
    new Schema\Generator\GenerateModifiers(),       // Generate changes from schema modifiers
    new Schema\Generator\ValidateEntities(),        // Validate all entity schemas
    new Schema\Generator\RenderTables(),            // Declare table schemas
    new Schema\Generator\RenderRelations(),         // Declare relation keys and indexes
    new Schema\Generator\RenderModifiers(),         // Render all schema modifiers
    new Schema\Generator\ForeignKeys(),             // Define foreign key constraints
    new Annotated\MergeIndexes(),                   // Register indexes from attributes
    new Schema\Generator\SyncTables(),              // Sync table changes to database
    new Schema\Generator\GenerateTypecast(),        // Typecast non-string columns
]);

$orm = $orm->with(schema: new \Cycle\ORM\Schema($schema));
```

> Make sure to point the class locator to the directory with your domain entities only, as the indexation operation
> is fairly expensive. Ensure that all entities are loadable by `composer autoload`.

The result of the schema builder is a compiled schema. The given schema can be cached in order to avoid expensive
calculations on each request.

> In the following sections, the term "update schema" refers to this schema compilation process.

Remove the `new Schema\Generator\SyncTables()` generator to disable automatic database synchronization. In production
environments, use database migrations instead of direct synchronization.

> Read more about [database migrations](/docs/en/database/migrations.md).

## Available Annotations

Once configured, you can use these attributes to define your entities:

### Entity Definitions

- `#[\Cycle\Annotated\Annotation\Entity]` - Defines an entity class
- `#[\Cycle\Annotated\Annotation\Embeddable]` - Defines an embeddable entity

### Column Definitions

- `#[\Cycle\Annotated\Annotation\Column]` - Defines a column mapping
- `#[\Cycle\Annotated\Annotation\GeneratedValue]` - Marks auto-generated fields (timestamps, UUIDs, auto-increment)

### Table Schema

- `#[\Cycle\Annotated\Annotation\Table\Index]` - Defines table indexes
- `#[\Cycle\Annotated\Annotation\Table\PrimaryKey]` - Defines composite primary keys
- `#[\Cycle\Annotated\Annotation\ForeignKey]` - Defines foreign key constraints without relations

### Relations

- `#[\Cycle\Annotated\Annotation\Relation\Embedded]` - Embeds an entity
- `#[\Cycle\Annotated\Annotation\Relation\BelongsTo]` - Defines belongs-to relationship
- `#[\Cycle\Annotated\Annotation\Relation\HasOne]` - Defines has-one relationship
- `#[\Cycle\Annotated\Annotation\Relation\HasMany]` - Defines has-many relationship
- `#[\Cycle\Annotated\Annotation\Relation\ManyToMany]` - Defines many-to-many relationship
- `#[\Cycle\Annotated\Annotation\Relation\RefersTo]` - Defines self-referencing or multiple relationships
- `#[\Cycle\Annotated\Annotation\Relation\Inverse]` - Defines inverse side of a relation

### Morphed (Polymorphic) Relations

- `#[\Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed]` - Child belongs to multiple parent types
- `#[\Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasOne]` - Parent has one polymorphic child
- `#[\Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasMany]` - Parent has many polymorphic children

### Table Inheritance

- `#[\Cycle\Annotated\Annotation\Inheritance\SingleTable]` - Single table inheritance strategy
- `#[\Cycle\Annotated\Annotation\Inheritance\JoinedTable]` - Joined table inheritance strategy
- `#[\Cycle\Annotated\Annotation\Inheritance\DiscriminatorColumn]` - Discriminator column for STI

> Read more about each attribute in the [Annotated Entities](/docs/en/annotated/entity.md)
> and [Relations](/docs/en/annotated/relations.md) documentation.
