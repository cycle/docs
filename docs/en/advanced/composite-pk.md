# Composite keys

A composite key is a combination of two or more columns in a table that can be used to uniquely identify each row in the
table when the columns are combined uniqueness is guaranteed, but when it takes individually it does not guarantee
uniqueness.

Sometimes more than one attributes are needed to uniquely identify an entity. A primary key that is made by the
combination of more than one attribute is known as a composite key.

Cycle ORM supports composite primary keys natively. For Cycle ORM composite keys of primitive data-types are supported,
even foreign keys as primary keys are supported.

## Declaration via Schema

You can use composite keys in your schema for Entity `Schema::PRIMARY_KEY` and for entity
relations `Relation::INNER_KEY`, `Relation::OUTER_KEY`.

```php
[
    User::class => [
        // ...
        Schema::PRIMARY_KEY => ['key1', 'key2'],                                // <===
        Schema::COLUMNS => [
            'key1' => 'field1',
            'key2' => 'field2',
            // ...
        ],
        Schema::RELATIONS => [
            'posts' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Post::class,
                Relation::SCHEMA => [
                    Relation::CASCADE => true,
                    Relation::INNER_KEY => ['key1', 'key2'],                    // <===
                    Relation::OUTER_KEY => ['parent_key1', 'parent_key2'],      // <===
                    Relation::ORDER_BY => ['key1' => 'asc', 'key2' => 'asc'],
                ],
            ],
        ],
    ],
];
```

## Declaration via annotations

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class Pivot
{
    #[Column(type: 'int', primary: true)]
    private int $postId;
    
    #[Column(type: 'int', primary: true)]
    private int $tagId;
}

#[Entity]
class Tag 
{
    #[HasMany(target: Pivot::class, outerKey: ['parent_key1', 'parent_key2'])]
    private array $posts;
}
```

## Select

To find records by composite PK you have to pass array of identifiers instead of single value.

```php
$repository = $orm->getRepository(Pivot::class);

$pivot = $repository->findByPK([
    1, // post id
    1 // tag id
]);
```
