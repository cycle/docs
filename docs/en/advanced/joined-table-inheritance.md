# Joined Table Inheritance

Joined Table Inheritance (JTI) creates separate database tables for each entity in the hierarchy. The only column that
appears in all tables is the identifier, which is used to join tables when loading entities.

The ORM provides the ability to map each class in the hierarchy to its own table. To achieve this, extend your parent
entity and declare relations/columns specific to each child.

<img width="826" alt="JoinedTable Inheritance" src="https://user-images.githubusercontent.com/773481/144869504-e3236f51-011e-448e-a02a-767880609bdb.png">

## Understanding Joined Table Inheritance

### What is a Foreign Key Join?

In JTI, each child entity has its own table that connects to the parent table through a **foreign key relationship**.
The child table's primary key serves double duty: it's both a primary key and a foreign key that references the parent
table.

For example, with `Person` as a parent and `Employee` as a child:

```sql
-- Parent table
CREATE TABLE persons
(
    id   INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255)
);

-- Child table - 'id' references parent
CREATE TABLE employees
(
    id     INT PRIMARY KEY, -- Primary key AND foreign key
    salary INT,
    FOREIGN KEY (id) REFERENCES persons (id)
);
```

When you load an `Employee`, Cycle ORM automatically joins both tables to retrieve all data.

### How JTI Works

When you save an entity, Cycle ORM:

1. Stores shared data in the parent table
2. Stores type-specific data in the child table
3. Uses the same ID value in both tables to link them

When you query entities, Cycle ORM:

1. Performs SQL JOINs between parent and child tables
2. Combines data from multiple tables into one object
3. Instantiates the appropriate PHP class based on which tables have data

### When to Use JTI

Use Joined Table Inheritance when your entities are **more different than similar**:

**✅ Good Use Cases:**

1. **Content Types** - Article (title, body, word_count), Video (title, duration, resolution, codec), Podcast (title,
   episode_number, audio_url)
2. **Payment Methods** - CreditCard (card_number, cvv, expiry), BankTransfer (routing_number, account_number),
   Cryptocurrency (wallet_address, blockchain_id)
3. **Shipments** - Ground (vehicle_type, driver_id), Air (flight_number, airline_code), Freight (container_type,
   cargo_manifest)

**❌ Avoid JTI When:**

- Entities share most of their columns (JOIN overhead not worth it)
- You frequently query across all entity types together
- Query performance is more critical than storage efficiency

## Table of Contents

- [Understanding Joined Table Inheritance](#understanding-joined-table-inheritance)
    - [What is a Foreign Key Join?](#what-is-a-foreign-key-join)
    - [How JTI Works](#how-jti-works)
    - [When to Use JTI](#when-to-use-jti)
- [Definition via Attributes](#definition-via-attributes)
    - [Attribute Specification](#attribute-specification)
    - [Basic Example](#basic-example)
    - [Custom Parent Keys](#custom-parent-keys)
    - [Multi-Level Inheritance](#multi-level-inheritance)
    - [Foreign Key Configuration](#foreign-key-configuration)
- [Schema Definition](#schema-definition)
- [Querying Entities](#querying-entities)
- [Important Considerations](#important-considerations)

## Definition via Attributes

### Attribute Specification

**`#[JoinedTable]` Attribute**

Applied to child entity classes to indicate they use Joined Table Inheritance.

| Property   | Type                                 | Required | Default     | Description                                                                            |
|------------|--------------------------------------|----------|-------------|----------------------------------------------------------------------------------------|
| `outerKey` | `non-empty-string\|null`             | No       | Primary key | The parent entity's key column used for joining. Defaults to the parent's primary key. |
| `fkCreate` | `bool`                               | No       | `true`      | Whether to automatically create a foreign key constraint on the join column.           |
| `fkAction` | `'NO ACTION'\|'CASCADE'\|'SET NULL'` | No       | `'CASCADE'` | Foreign key action for both `onDelete` and `onUpdate` events.                          |

> **Note**
> Make sure the parent class properties are **not private** so child entities can inherit and access them.

### Basic Example

By default, child entities are joined to the parent via the primary key:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Inheritance\JoinedTable;

#[Entity]
class Person
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;
}

#[Entity]
#[JoinedTable] // Joined via Person.id = Employee.id
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[JoinedTable] // Joined via Person.id = Customer.id
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

This creates three tables:

- `persons`: Contains `id` and `name`
- `employees`: Contains `id` (FK to persons) and `salary`
- `customers`: Contains `id` (FK to persons) and `preferences`

> **Read more** about entity definition in the [Entity Attributes](../annotated/entity.md) section.

### Custom Parent Keys

Use the `outerKey` parameter to join via a different column than the primary key:

```php
#[Entity]
class Person
{
    #[Column(type: 'primary')]
    protected int $id;
    
    #[Column(type: 'int', unique: true)]
    protected int $fooId; // Alternative join column

    #[Column(type: 'string')]
    protected string $name;
}

#[Entity]
#[JoinedTable(outerKey: 'fooId')] // Join via Person.fooId = Customer.fooId
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}
```

In this example, the `customers` table's `fooId` column references the `persons` table's `fooId` column instead of the
primary key.

### Multi-Level Inheritance

Joined Table Inheritance supports multiple inheritance levels, with each child entity joined to its immediate parent:

```php
#[Entity]
class Person
{
    #[Column(type: 'primary')]
    protected int $id;

    #[Column(type: 'string')]
    protected string $name;
}

#[Entity]
#[JoinedTable] // Joined to Person
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[JoinedTable] // Joined to Employee (not Person)
class Executive extends Employee
{
    #[Column(type: 'int')]
    protected int $bonus;
}
```

This creates three tables:

- `persons`: Contains `id` and `name`
- `employees`: Contains `id` (FK to persons) and `salary`
- `executives`: Contains `id` (FK to employees) and `bonus`

When loading an `Executive`, Cycle ORM performs joins across all three tables.

### Foreign Key Configuration

Control foreign key creation and behavior with the `fkCreate` and `fkAction` parameters:

```php
#[Entity]
#[JoinedTable(
    fkCreate: true,      // Create foreign key constraint (default)
    fkAction: 'CASCADE'  // ON DELETE CASCADE, ON UPDATE CASCADE (default)
)]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

#[Entity]
#[JoinedTable(
    fkCreate: true,
    fkAction: 'SET NULL' // ON DELETE SET NULL, ON UPDATE SET NULL
)]
class Customer extends Person
{
    #[Column(type: 'json')]
    protected array $preferences;
}

#[Entity]
#[JoinedTable(
    fkCreate: false // Don't create foreign key constraint
)]
class Guest extends Person
{
    #[Column(type: 'datetime')]
    protected \DateTimeImmutable $visitedAt;
}
```

**Foreign Key Actions:**

- `'CASCADE'` (default): Delete/update child rows when parent is deleted/updated
- `'SET NULL'`: Set foreign key to NULL when parent is deleted/updated (requires nullable column)
- `'NO ACTION'`: Prevent parent deletion/update if children exist

> **Note:** When `fkCreate` is `true`, Cycle ORM automatically creates a unique index on the parent key column and
> establishes the foreign key relationship.

## Schema Definition

You can configure Joined Table Inheritance programmatically without attributes by defining the `SchemaInterface::PARENT`
and `SchemaInterface::PARENT_KEY` segments for child entities:

```php
use Cycle\ORM\SchemaInterface;

$schema = [
    'person' => [
        // ... other schema configuration
    ],
    'employee' => [
        // ... other schema configuration
        SchemaInterface::PARENT => Person::class,      // Parent entity
        SchemaInterface::PARENT_KEY => 'id',           // Join column
    ],
    'customer' => [
        // ... other schema configuration
        SchemaInterface::PARENT => Person::class,
        SchemaInterface::PARENT_KEY => 'fooId',        // Custom join column
    ],
    'executive' => [
        // ... other schema configuration
        SchemaInterface::PARENT => Employee::class,    // Multi-level inheritance
        SchemaInterface::PARENT_KEY => 'id',
    ]
];
```

The `PARENT` specifies the parent entity class, and `PARENT_KEY` defines the column used for joining (defaults to the
parent's primary key if not specified).

## Querying Entities

When querying a parent entity repository, Cycle ORM automatically performs the necessary joins and returns instances of
the appropriate entity type:

```php
// Performs LEFT JOINs and returns Person, Employee, Customer, and Executive instances
$people = $orm->getRepository(Person::class)->findAll();

foreach ($people as $person) {
    if ($person instanceof Employee) {
        echo "Salary: " . $person->salary;
    } elseif ($person instanceof Customer) {
        echo "Preferences: " . json_encode($person->preferences);
    }
}
```

Child entity repositories work efficiently with automatic joins:

```php
// Automatically joins persons and employees tables
$employees = $orm->getRepository(Employee::class)->findAll();

// Multi-level join: persons → employees → executives
$executives = $orm->getRepository(Executive::class)->findAll();
```

You can optimize queries by selecting only necessary fields:

```php
// Only load parent fields (no child joins)
$people = $orm->getRepository(Person::class)
    ->select()
    ->columns(['id', 'name'])
    ->fetchAll();
```

> **Read more** about querying entities in the [Select Queries](../query-builder/basic.md) section.

## Important Considerations

### Limitations

- **Custom Repositories**: You cannot assign custom repositories to child entities. All queries must go through the
  entity's repository, but joins are handled automatically.
- **Scopes**: Custom scopes cannot be assigned to child entities in JTI hierarchies.

### Property Visibility

Parent class properties must be **protected** or **public**, not **private**. Private properties cannot be inherited:

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

### Primary Key Transformation

When a child entity extends a parent with a generated primary key (e.g., auto-increment), the child's primary key column
is **not** generated. It becomes a regular integer column that references the parent:

```php
#[Entity]
class Person
{
    #[Column(type: 'primary')] // Auto-increment primary key
    protected int $id;
}

#[Entity]
#[JoinedTable]
class Employee extends Person
{
    // Inherited 'id' becomes a non-generated primary key 
    // (foreign key to Person.id)
}
```

This means:

- `Person.id` uses `type: 'primary'` (auto-increment)
- `Employee.id` uses `type: 'integer'` with `primary: true` (foreign key)
- The same applies to `bigPrimary` → `bigInteger`

### Database Schema

The resulting database schema separates data across tables:

**Parent Table (persons):**

```sql
CREATE TABLE persons
(
    id   INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

**Child Table (employees):**

```sql
CREATE TABLE employees
(
    id     INT PRIMARY KEY,
    salary INT NOT NULL,
    FOREIGN KEY (id) REFERENCES persons (id) ON DELETE CASCADE ON UPDATE CASCADE
);
```

### Performance Considerations

**Advantages:**

- Normalized data structure (no wasted space)
- Easy to add child-specific columns without affecting other tables
- Can enforce NOT NULL constraints on child-specific columns
- Better for hierarchies with significantly different structures

**Disadvantages:**

- Requires joins for queries (potentially slower than Single Table Inheritance)
- More complex schema
- Multiple tables to manage

Choose JTI when:

- Child entities have significantly different structures
- You need to enforce data integrity with NOT NULL constraints
- Storage efficiency is important
- You have many child-specific columns that would create a very wide table in STI

> **See also:** For simpler hierarchies where all entities share similar structures,
> consider [Single Table Inheritance](./single-table-inheritance.md) instead.

### Combining with Single Table Inheritance

You can combine both inheritance strategies in a single hierarchy:

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

// STI children - share persons table
#[Entity]
#[SingleTable]
class Guest extends Person
{
    #[Column(type: 'datetime', nullable: true)]
    protected ?\DateTimeImmutable $visitedAt = null;
}

// JTI child - uses separate table
#[Entity]
#[JoinedTable]
class Employee extends Person
{
    #[Column(type: 'int')]
    protected int $salary;
}

// JTI child of JTI parent
#[Entity]
#[JoinedTable]
class Manager extends Employee
{
    #[Column(type: 'string')]
    protected string $department;
}
```

In this example:

- `Person` and `Guest` share the `persons` table (STI)
- `Employee` has its own `employees` table (JTI from Person)
- `Manager` has its own `managers` table (JTI from Employee)
