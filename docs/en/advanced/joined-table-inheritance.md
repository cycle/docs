# Joined Table Inheritance

JTI creates separate tables for each entity. The only column that repeatedly appears in all the tables is the
identifier, which will be used for joining them when needed.

The ORM provides the ability to map each class in the hierarchy to its table. In order to achieve that you must extend
your parent entity and declare relations/columns specific to the child.

<img width="826" alt="JoinedTable Inheritance" src="https://user-images.githubusercontent.com/773481/144869504-e3236f51-011e-448e-a02a-767880609bdb.png">

## Definition via attributes

> **Note**
> Make sure the parent class properties are **not private**.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Inheritance\JoinedTable;

#[Entity]
class Person
{
    #[Column(primary: true)]
    protected int $id;
    
    #[Column]
    protected int $fooId;

    #[Column(type: 'string')]
    protected string $name;
}

#[Entity]
#[JoinedTable]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[JoinedTable(outerKey: 'fooId')]
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}

#[Entity]
#[JoinedTable]
class Executive extends Employee
{
    #[Column(type: 'int')]
    protected int $bonus;
}
```

By default, all entities with `#[JoinedTable]` attribute will behavior as joined table entities and will be connected
with root entity via primary column. If you need to change parent key columns, you need to pass `outerKey` via
attribute `#[JoinedTable(outerKey: 'fooId')]`

## Schema definition

You can configure Joined table inheritance without attributes by defining `SchemaInterface::PARENT`
and `SchemaInterface::PARENT_KEY` segments for child entities.

```php
use Cycle\ORM\SchemaInterface;

$schema = [
    'person' => [
        ...
    ],
    'employee' => [
        ...,
        SchemaInterface::PARENT => Person::class,
        SchemaInterface::PARENT_KEY => 'id',
    ],
    'customer' => [
        ...,
        SchemaInterface::PARENT => Person::class,
        SchemaInterface::PARENT_KEY => 'fooId',
    ],
    'executive' => [
        ...,
        SchemaInterface::PARENT => Person::class,
        SchemaInterface::PARENT_KEY => 'id',
    ]
]
```

## Querying

You have to remember that fetching entities from the repository might return any of child entity:

```php
// persons, employees and customers will be returned
$people = $orm->getRepository(Person::class)->findAll();
```

You are currently not allowed to assign custom repositories or scopes to child entities. However, you can use `type` in
your queries to pre-filter the selection.
