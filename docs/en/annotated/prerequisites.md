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

- `#[Entity]` - Defines an entity class
- `#[Embeddable]` - Defines an embeddable entity

### Column Definitions

- `#[Column]` - Defines a column mapping
- `#[GeneratedValue]` - Marks auto-generated fields (timestamps, UUIDs, auto-increment)

### Table Schema

- `#[Table\Index]` - Defines table indexes
- `#[Table\PrimaryKey]` - Defines composite primary keys
- `#[ForeignKey]` - Defines foreign key constraints without relations

### Relations

- `#[Relation\Embedded]` - Embeds an entity
- `#[Relation\BelongsTo]` - Defines belongs-to relationship
- `#[Relation\HasOne]` - Defines has-one relationship
- `#[Relation\HasMany]` - Defines has-many relationship
- `#[Relation\ManyToMany]` - Defines many-to-many relationship
- `#[Relation\RefersTo]` - Defines self-referencing or multiple relationships
- `#[Relation\Inverse]` - Defines inverse side of a relation

### Morphed (Polymorphic) Relations

- `#[Relation\Morphed\BelongsToMorphed]` - Child belongs to multiple parent types
- `#[Relation\Morphed\MorphedHasOne]` - Parent has one polymorphic child
- `#[Relation\Morphed\MorphedHasMany]` - Parent has many polymorphic children

### Table Inheritance

- `#[Inheritance\SingleTable]` - Single table inheritance strategy
- `#[Inheritance\JoinedTable]` - Joined table inheritance strategy
- `#[Inheritance\DiscriminatorColumn]` - Discriminator column for STI

> Read more about each attribute in the [Annotated Entities](/docs/en/annotated/entity.md)
> and [Relations](/docs/en/annotated/relations.md) documentation.
