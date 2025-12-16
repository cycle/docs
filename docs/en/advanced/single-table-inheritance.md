# Single Table Inheritance

Single Table Inheritance (STI) stores entities from different classes with a common ancestor in a single database table.
This creates one table for the entire class hierarchy, with a discriminator column to differentiate between entity
types.

The ORM provides the ability to store multiple model variations inside one table. To achieve this, extend your parent
entity and declare relations/columns specific to each child.

<img width="826" alt="Single Inheritance" src="https://user-images.githubusercontent.com/773481/144869132-f7f32a00-aa84-4e70-8ccc-3c7e26860bba.png">

## Understanding Single Table Inheritance

### What is a Discriminator?

In STI, the **discriminator column** is a special database column that identifies which type of entity each row
represents. Think of it as a "type tag" that tells Cycle ORM which PHP class to instantiate when loading data.

For example, in a `persons` table with the discriminator column `type`:

- Rows with `type = 'employee'` are loaded as `Employee` objects
- Rows with `type = 'customer'` are loaded as `Customer` objects
- Rows with `type = 'admin'` are loaded as `Admin` objects

This allows one table to store multiple entity types while Cycle ORM automatically handles the conversion.

### How STI Works

When you save an entity, Cycle ORM:

1. Stores all data in a single table
2. Automatically sets the discriminator column value based on the entity type
3. Uses NULL for columns that don't apply to that specific entity type

When you query entities, Cycle ORM:

1. Reads the discriminator column value
2. Instantiates the correct PHP class based on that value
3. Populates only the relevant properties for that entity type

### When to Use STI

Use Single Table Inheritance when your entities are **more similar than different**:

**✅ Good Use Cases:**

1. **User Types** - Admin, Customer, Guest with shared authentication but different permissions
2. **Notifications** - Email, SMS, Push with common tracking but different delivery methods
3. **Content Items** - Post, Page, Comment sharing publication metadata but different layouts

**❌ Avoid STI When:**

- Entities have many type-specific columns (creates too many nullable columns)
- Child types have completely different data structures
- You need strict NOT NULL constraints on child-specific fields

## Definition via Attributes

### Attribute Specification

**`#[SingleTable]` Attribute**

Applied to child entity classes to indicate they participate in Single Table Inheritance.

| Property | Type                                                 | Required | Default     | Description                                                                       |
|----------|------------------------------------------------------|----------|-------------|-----------------------------------------------------------------------------------|
| `value`  | `string\|int\|float\|\Stringable\|\BackedEnum\|null` | No       | Entity role | The discriminator value for this entity type. Defaults to the entity's role name. |

**`#[DiscriminatorColumn]` Attribute**

Applied to the root entity class to specify the discriminator column name.

| Property | Type               | Required | Default | Description                                                         |
|----------|--------------------|----------|---------|---------------------------------------------------------------------|
| `name`   | `non-empty-string` | **Yes**  | -       | The database column name that stores the entity type discriminator. |

> **Note**
> Make sure the parent class properties are **not private** so child entities can inherit and access them.

### Basic Example

The discriminator column attribute is **mandatory** on the root entity. By default, the discriminator value for each
entity is its role name:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Inheritance\SingleTable;
use Cycle\Annotated\Annotation\Inheritance\DiscriminatorColumn;

#[Entity]
#[DiscriminatorColumn(name: 'type')] // Required on root entity
class Person
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;
    
    #[Column(type: 'string')]
    protected string $type; // Discriminator column
}

#[Entity]
#[SingleTable] // Discriminator value: "employee" (entity role)
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[SingleTable] // Discriminator value: "customer" (entity role)
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

> **Read more** about entity definition in the [Entity Attributes](../annotated/entity.md) section.

### Custom Discriminator Values

You can specify custom discriminator values instead of using the default entity role:

```php
#[Entity]
#[SingleTable(value: 'super_customer')]
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

The `value` parameter accepts various types including `BackedEnum` for type-safe discriminator values:

```php
enum PersonType: string
{
    case Employee = 'emp';
    case Customer = 'cust';
}

#[Entity]
#[SingleTable(value: PersonType::Employee)]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[SingleTable(value: PersonType::Customer)]
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

### Multi-Level Inheritance

Single Table Inheritance supports multiple levels of inheritance. All entities in the hierarchy share the same table:

```php
#[Entity]
#[DiscriminatorColumn(name: 'type')]
class Person
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;

    #[Column(type: 'string')]
    protected string $type;
}

#[Entity]
#[SingleTable]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

// CEO inherits from Employee, which inherits from Person
#[Entity]
#[SingleTable] // Discriminator value: "ceo"
class Ceo extends Employee
{
    #[Column(type: 'int')]
    protected int $stocks;
    
    #[Column(type: 'int')]
    protected int $bonus;
}
```

In this example, the `persons` table will contain columns for all three entity types: `id`, `name`, `type`, `salary`,
`stocks`, and `bonus`.

## Schema Definition

You can configure Single Table Inheritance programmatically without attributes by defining the
`SchemaInterface::CHILDREN` and `SchemaInterface::DISCRIMINATOR` segments for the root entity:

```php
use Cycle\ORM\SchemaInterface;

$schema = [
    'person' => [
        // ... other schema configuration
        SchemaInterface::CHILDREN => [
            // discriminator value => Entity class
            'employee' => Employee::class,
            'foo_customer' => Customer::class,
            'ceo' => Ceo::class,
        ],
        SchemaInterface::DISCRIMINATOR => 'type' // Discriminator column name
    ],
    'employee' => [
        // ... other schema configuration
        SchemaInterface::ENTITY => Employee::class,
    ],
    'customer' => [
        // ... other schema configuration
        SchemaInterface::ENTITY => Customer::class,
    ],
    'ceo' => [
        // ... other schema configuration
        SchemaInterface::ENTITY => Ceo::class,
    ]
];
```

The `CHILDREN` array maps discriminator values to entity class names. The `DISCRIMINATOR` defines the column name used
to identify entity types.

## Querying Entities

When querying a parent entity repository, Cycle ORM automatically returns instances of the appropriate child entity
based on the discriminator value:

```php
// Returns Person, Employee, Customer, and Ceo instances
// based on the 'type' discriminator column value
$people = $orm->getRepository(Person::class)->findAll();

foreach ($people as $person) {
    if ($person instanceof Employee) {
        echo "Salary: " . $person->salary;
    } elseif ($person instanceof Customer) {
        echo "Preferences: " . json_encode($person->preferences);
    }
}
```

You can filter by discriminator value in queries:

```php
// Get only employees
$employees = $orm->getRepository(Person::class)
    ->select()
    ->where('type', 'employee')
    ->fetchAll();

// Using custom discriminator values
$customers = $orm->getRepository(Person::class)
    ->select()
    ->where('type', 'super_customer')
    ->fetchAll();
```

Child entity repositories work as expected:

```php
// Returns only Employee instances
$employees = $orm->getRepository(Employee::class)->findAll();
```

> **Read more** about querying entities in the [Select Queries](../query-builder/basic.md) section.

## Important Considerations

### Limitations

- **Custom Repositories**: You cannot assign custom repositories to child entities. All queries must go through the
  parent entity's repository or use discriminator filtering.
- **Scopes**: Custom scopes cannot be assigned to child entities in STI hierarchies.
- **Column Visibility**: All columns for all child entities exist in the same table. Nullable columns should be used for
  child-specific fields.

### Property Visibility

Parent class properties must be **protected** or **public**, not **private**. Private properties cannot be inherited and
will cause issues:

```php
// ✅ Correct
#[Entity]
class Person
{
    #[Column(type: 'primary')]
    protected int $id; // Protected - accessible to children
}

// ❌ Incorrect
#[Entity]
class Person
{
    #[Column(type: 'primary')]
    private int $id; // Private - NOT accessible to children
}
```

### Database Schema

The resulting database table contains:

- All columns from the parent entity
- All columns from all child entities
- The discriminator column

Child-specific columns should generally be nullable since not all rows will use them:

```php
#[Entity]
#[SingleTable]
class Employee extends Person
{
    #[Column(type: 'int', nullable: true)] // Nullable for non-employee rows
    protected ?int $salary = null;
}
```

### Performance Considerations

**Advantages:**

- Fast queries (no joins required)
- Simple schema structure
- Easy to add new child types

**Disadvantages:**

- Table can become wide with many child types
- Wasted space from unused nullable columns
- Cannot enforce NOT NULL constraints on child-specific columns

Choose STI when:

- Child entities share most of their structure
- You have a limited number of child types
- Query performance across the hierarchy is important
- The hierarchy is relatively shallow

> **See also:** For hierarchies with significantly different structures,
> consider [Joined Table Inheritance](./joined-table-inheritance.md) instead.
