# Single Table Inheritance

The entities from different classes with a common ancestor are placed in a single table. STI creates one table for each
class hierarchy.

The ORM provides the ability to store multiple model variations inside one table. In order to achieve that you must
extend your parent entity and declare relations/columns specific to the child.

<img width="826" alt="Single Inheritance" src="https://user-images.githubusercontent.com/773481/144869132-f7f32a00-aa84-4e70-8ccc-3c7e26860bba.png">

## Definition via attributes

We can define the strategy we want to use by adding the `#[SingleTable]` attribute to a subclass.

 > **Note**
 > Make sure the parent class properties are **not private**.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Inheritance\SingleTable;
use Cycle\Annotated\Annotation\Inheritance\DiscriminatorColumn;

#[Entity]
#[DiscriminatorColumn(name: 'type')]
class Person
{
    #[Column(type: 'primary', primary: true)]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;

    #[Column(type: 'string')]
    protected string $type;
}

#[Entity]
#[SingleTable] // discriminator value: employee
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[SingleTable] // discriminator value: customer
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}

#[Entity]
#[SingleTable]
class Ceo extends Employee
{
    #[Column(type: 'int')]
    public int $stocks;
}
```

Since the records for all entities will be in the same table, CycleORM needs a way to differentiate between them. By
default, this is done through a discriminator column called `type` that has the name of the entity role as a value. The
discriminator column `#[DiscriminatorColumn]` attribute is mandatory.

Next, we need to tell CycleORM what value each subclass entity will have for the `type` column. By default, CycleORM
uses entity role as a discriminator value, but you can change it by passing a value via `#[SingleTable]` attribute:

```php
#[Entity]
#[SingleTable(value: 'super_customer')] // discriminator value: super_customer
class Customer extends Person
{
    //...
}
```

## Schema definition

You can configure Single table inheritance without attributes by defining `SchemaInterface::CHILDREN`
and `SchemaInterface::DISCRIMINATOR` segments for root entity.

```php
use Cycle\ORM\SchemaInterface;

$schema = [
    'person' => [
        ...,
        SchemaInterface::CHILDREN => [
            // discriminator value => Entity class
            'employee' => Employee::class,
            'foo_customer' => Customer::class,
            'ceo' => Ceo::class,
        ],
        SchemaInterface::DISCRIMINATOR => 'type'
    ],
    'employee' => [
        ...,
        SchemaInterface::ENTITY => Employee::class,
    ],
    'customer' => [
        ...,
        SchemaInterface::ENTITY => Customer::class,
    ],
    'ceo' => [
        ...,
        SchemaInterface::ENTITY => Ceo::class,
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
