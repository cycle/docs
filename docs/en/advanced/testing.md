# Testing

Cycle ORM attempts to simplify the testing of your application by providing well-isolated interfaces to load and persist
your entities.

## Classic Mock Approach

The first approach would involve mocking instances of entity repositories and `EntityManagerInterface`, for example the
given code can be well tested without ORM initialization:

```php
public function addOrder(User $user, Order $order, \Cycle\ORM\EntityManagerInterface $entityManager)
{
    $user->orders->add($order);
    $entityManager->persist($user)->run();
}
```

Mock `EntityManagerInterface` to check the state of the entity after the method call. Additionally, you can mock custom
entity repositories:

```php
class MyService
{
    public function __construct(
        private UserRepository $users
    ) {
    }

    public function disableUser(int $id, \Cycle\ORM\EntityManagerInterface $entityManager): void
    {
        $user = $this->users->findByPK($id);
        $user->status = 'disabled';
        $entityManager->persist($user)->run();
    }
}
```

> You can also `run` transactions outside of your service method if you wish to group multiple persist operations together.

## Mocking The Database

Another, more complex and slower, approach would involve creating a test database and running your service code using
more realistic scenarios.

In order to achieve that you must construct your own `DatabaseManager` instance and replace the desired database
connection with the required driver (for example SQLite):

```php
use Cycle\ORM;
use Cycle\Database;
use Cycle\Database\Config;

$dbal = new Database\DatabaseManager(
    new Config\DatabaseConfig([
        'default' => 'default',
        'databases' => [
            'default' => [
                'connection' => 'sqlite'
            ]
        ],
        'connections' => [
            'sqlite' => new Config\SQLiteDriverConfig(
                connection: new Config\SQLite\MemoryConnectionConfig(),
                queryCache: true,
            ),
        ]
    ])
);

$orm = new ORM\ORM(new ORM\Factory($dbal));

// you can use already calculated database schema
$orm = $orm->with(schema: new ORM\Schema($cachedSchema));
```

> Attention, you would not be able to test if your database constraints operate properly using SQLite. Use the dedicated database instance.

You can clean your database state after each iteration using schema introspection approach:

```php
public function tearDown()
{
    $db = $this->dbal->database('default');

    // delete all FKs first
    foreach ($database->getTables() as $table) {
        $schema = $table->getSchema();
        foreach ($schema->getForeignKeys() as $foreign) {
            $schema->dropForeignKey($foreign->getColumns());
        }

        $schema->save(\Cycle\Database\Driver\HandlerInterface::DROP_FOREIGN_KEYS);
    }

    // delete tables
    foreach ($database->getTables() as $table) {
        $schema = $table->getSchema();
        $schema->declareDropped();
        $schema->save();
    }
}
```

> Attention, such tests will run much slower than with mocked repositories and transactions.
