# Active Entities

## Defining Entities

To work with the Cycle Active Record, you need to extend the entity class with the `ActiveRecord`.

> Note
> Below are examples of defining entities using attributes (see [Annotated Entities](/docs/en/annotated/entity.md)),
> but you are still free to define entities in any other way.

:::: tabs

::: tab Strict

**Strict Approach:**

In the **strict** approach, you define your entity with private properties and provide public getter and setter methods to access and modify the properties.

This approach encapsulates the entity's internal state and provides better control over how the properties are accessed and modified.

```php
<?php

declare(strict_types=1);

namespace App\Entities;

use Cycle\ActiveRecord\ActiveRecord;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\CreatedAt;

#[CreatedAt(field: 'createdAt')]
#[Entity(table: 'users')]
class User extends ActiveRecord
{
    #[Column(type: 'bigInteger', primary: true, typecast: 'int')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    #[Column(type: 'string', unique: true)]
    private string $email;

    private \DatetimeImmutable $createdAt;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public static function create(string $name, string $email): static
    {
        return static::make([
            'name' => $name,
            'email' => $email,
        ]);
    }

    public function id(): int
    {
        return $this->id;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function changeName(string $name): void
    {
        $this->name = $name;
    }

    public function email(): string
    {
        return $this->email;
    }

    public function changeEmail(string $email): void
    {
        $this->email = $email;
    }
}
```

:::

::: tab Fluent

**Fluent Approach:**

In the **fluent** approach, you define your entity with public properties, allowing direct access to the properties without the need for explicit getter and setter methods.

This approach leads to more concise and readable code, especially when dealing with simple entities.

```php
<?php

declare(strict_types=1);

namespace App\Entities;

use Cycle\ActiveRecord\ActiveRecord;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\CreatedAt;

#[CreatedAt(field: 'createdAt')]
#[Entity(table: 'users')]
class User extends ActiveRecord
{
    #[Column(type: 'bigInteger', primary: true, typecast: 'int')]
    public int $id;

    #[Column(type: 'string')]
    public string $name;

    #[Column(type: 'string', unique: true)]
    public string $email;

    public \DatetimeImmutable $createdAt;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
}
```

:::

::: tab Extended

In this example, we do not focus on whether the fields in the entity are private or public. What is important here is:

- Value Objects are used instead of primitive types. See [Column wrappers](/docs/en/advanced/column-wrappers.md) to learn more.
- We prohibit the use of the constructor.

```php
#[CreatedAt(field: 'createdAt')]
#[Entity(table: 'users')]
class User extends ActiveRecord
{
    #[Column(type: 'bigInteger', primary: true, typecast: 'uuid')]
    public UuidInterface $id;

    #[Column(type: 'string', typecast: UserName::class)]
    public UserName $name;

    #[Column(type: 'string', unique: true, typecast: UserEmail::class)]
    public UserEmail $email;

    public \DatetimeImmutable $createdAt;

    private function __construct() {}

    public static function create(string $name, string $email): static
    {
        return static::make([
            'name' => $name,
            'email' => $email,
        ]);
    }
}
```

A simple Value Object sample:

```php UserName.php
// Stringable interface is used to provide a string value for Database
class UserName implements \Stringable
{
    // ... constructor and __toString method

    // Will be used by the User
    public static function create(string $value): self
    {
        self::validate($value);
        return new self($value);
    }

    // Will be used by ORM that's why validation is not needed
    public static function typecast(string $value): self
    {
        return new self($value);
    }
}
```

### Explanation

Using the constructor when creating entities can lead to [undesirable effects](https://spiral.dev/blog/cycle-orm-hungry-for-relations).
It's recommended to use `ORMInterface::make()` instead of the constructor.
To be friendlier, `ActiveRecord` provides the `make()` method directly on the entity.

```php
$user = User::make([
   'name' => 'John Doe',
   'email' => 'johndoe@example.com',
]);
```

However, this is still not convenient enough. The recommended approach is as follows:

- Use Value Objects for fields that may also validate the data.
  This approach adds a lot of type safety, and you can ensure that the data is always valid.
- Create a static factory method that will accept all necessary fields:
  ```php
  public static function create(UserName $name, UserEmail $email): static
  {
      return static::make([
          'name' => $name,
          'email' => $email,
      ]);
  }
  ```
- Make the constructor private to prohibit creating entities via `new`.
- You can inherit from the `ActiveRecord` class to extend this to all Active Entities:
  ```php
  abstract class AR extends \Cycle\ActiveRecord\ActiveRecord
  {
      final private function __construct() {}
  }

  class User extends AR
  {
      // ...
  }
  ```

:::

::::


## Creating Entities

Once the entities are described, we can start working with them.

Using constructors is not recommended, as it can lead to [undesirable effects](https://spiral.dev/blog/cycle-orm-hungry-for-relations), but it's still possible to use them.
Look at the [Extended Defining Entities](#defining-entities) section for more information.

### Using `make()` Method

The `make()` method is a static factory method that accepts an array of properties and returns a new instance of the entity.

```php
$user = User::make([
    'name' => $name,
    'email' => $email,
]);
```

## Persisting Entities

To save the entity to the database, you can use the `save()` method:
The `save()` method returns a result object that contains information about the operation.

```php
$result = $user->save();

// Check if the operation was successful
$result->isSuccess();
```

If you prefer to have an exception thrown when the operation fails, you can use the `saveOrFail()` method:

```php
$user->saveOrFail();
```


## Querying Entities

Entities that extends `ActiveRecord` class includes powerful Cycle-ORM Query Builder
allowing you to fluently query the database table associated with the Entity.

The process of querying data usually takes the following three steps:

1. Create a new query object by calling the `ActiveRecord::query()` method;
2. Build the query object by calling query building methods;
3. Call a query method to retrieve data in terms of Active Record instances.

### Building Queries

The `ActiveRecord::query()` method returns a new instance of the `ActiveQuery` class,
which is a wrapper around the query builder provided by Cycle ORM.
It means that you can use all the methods provided by the [query builder](/docs/en/query-builder/basic.md) and additionally some methods provided by the related [`ActiveQuery` class](/docs/en/active-record/active-query.md).

```php
// Query all the dog people
$dogsGuys = User::query()
    ->where('likes_dogs', true)
    ->fetchAll();

// Query 10 users who have a dog named Rex
$rexOwners = User::query()
    ->load('pet', ['where' => ['type' => 'dog', 'name' => 'Rex']])
    ->limit(10)
    ->fetchAll();

// Query user by id
$user = User::findByPK($id);
```

## Deleting Entities

To delete a single record, first retrieve the Active Record entity and then call the `delete()` or `deleteOrFail()` method.

```php
User::findByPK($id)->delete();

Post::findByPK($id)->deleteOrFail();
```

## Transactions

Each call to the `save()`, `delete()`, `deleteOrFail()`, or `saveOrFail()` method
runs at least one transaction to the database.
However, you may need to save multiple distinct entities in a single transaction.
To achieve this, you can use two methods: `::transact()` and `::groupActions()`.

### ActiveRecord::transact()

The `::transact()` method will immediately open a transaction and complete it along with the callback.
The transaction will be rolled back if an exception is thrown from the function; otherwise, it will be committed.

All the ORM operations within the callback will be executed in the opened transaction without collecting.
If you need to collect ORM operations and execute them in a separated inner transaction,
use `::groupActions()` within the callback.

If you call this method from a child class, the child database connection will be used for the transaction.
If you call this method from the `ActiveRecord` class, the default database connection will be used.

```php
$result = ActiveRecord::transact(
    static function (DatabaseInterface $db, EntityManagerInterface $em) use ($user, $account, $post): int {
        $db->query('DELETE FROM users');

        // All the ORM actions will be executed right away in the opened transaction using isolated UoW
        $user->save();
        $account->saveOrFail();
        $post->delete();

        // EM executes action right away, you don't need to call $em->run()
        $em->persist(new Post('Title', 'Content'));

        return 42;
    }
);
echo $result; // 42
```

### ActiveRecord::groupActions()

The `::groupActions()` method differs from `::transact()` in that:
- Only ActiveRecord write operations (saving and deleting entities) are captured and grouped into the transaction.
- DBAL and QueryBuilder calls are executed immediately, as if `::groupActions()` was not called.
- The transaction is opened after the callback is executed.

In other words, all operations on entities within the function are placed into a single Unit of Work of the EntityManager and executed upon exiting the function.

You can get the EntityManager as the first parameter of the passed function to perform actions with other ORM entities that are not converted to ActiveRecord.
To configure the transaction behavior of the Entity Manager, you can use the second parameter:

```php
ActiveRecord::groupActions(
    function (EntityManagerInterface $em) use ($users, $user, $account, $post) {
        array_walk($users, fn ($user) => $user->save());
        $user->save();
        $post->delete();
        $em->persist($account);
    },
    TransactionMode::Ignore,
);
```

The `TransactionMode` enum provides the following options:

- `TransactionMode::OpenNew` (default) starts a new transaction when the Entity Manager is running after the callback.
- `TransactionMode::Current` uses the current transaction and throws an exception if there is no active transaction.
- `TransactionMode::Ignore` does nothing about transactions. If there is an active transaction,
  it won't be committed or rolled back.

You can nest the call to `::groupActions()` inside the call to `::transact()` to avoid creating unnecessary nested transactions.
Nested calls to `::groupActions()` will use separated Unit of Works, but transactions will be reused according to the given mode.

```php
User::transact(function (DatabaseInterface $dbal, EntityManagerInterface $em) use ($user, $account, $post): void {
    $dbal->query('DELETE FROM users');

    User::groupActions(function () use ($user, $account, $post) {
        $user->save();
        $account->save();
        $post->delete();
    }, TransactionMode::Current);
});
```
