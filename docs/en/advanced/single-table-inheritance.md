# Single Table Inheritance
The ORM provides the ability to store multiple model variations inside one table. In order to achieve that you must extend your parent entity
and declare relations/columns specific to the child.

## Definition

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity]
#[DiscriminatorColumn(name: 'type')] // Discriminator column (required)
class Person
{
    #[Column(type: 'primary', primary: true)]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;
}

#[Entity]
#[InheritanceSingleTable]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[InheritanceSingleTable(value: 'foo_customer')]
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

You can store an `Employee` the same way as the parent entity.

> Note, ORM will create a special Discriminator column in your entity table, `type`, in which the child id (Discriminator value) will be stored.

## Querying
You have to remember that fetching entities from the repository might return any of child entity:

```php
// persons, employees and customers will be returned
$people = $orm->getRepository(Person::class)->findAll();
```

You are currently not allowed to assign custom repositories or scopes to child entities. However, you can use `type` in your queries to pre-filter the selection.
