# Testing
Cycle ORM attempts to simplify the testing of your application by providing well-isolated interfaces to load and persist your entities.

## Classic Mock Approach
The first approach would involve mocking instances of entity repositories and `TransactionInterface`, for example the given code can be well tested without ORM initialization:

```php
public function addOrder(User $u, Order $o, \Cycle\ORM\TransactionInterface $t)
{
    $u->orders->add($o);
    $t->persist($u)->run();
}
```

Mock `TransactionInterface` to check the state of the entity after the method call. Additionally, you can mock custom entity repositories:

```php
class MyService
{
    private $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function disableUser($id, \Cycle\ORM\TransactionInterface $t)
    {
        $u = $this->users->findByPK($id);
        $u->status = 'disabled';
        $t->persist($u)->run();
    }
}
```

> You can also `run` transactions outside of your service method if you wish to group multiple persist operations together.

## Mocking The Database
Another, more complex and slower, approach would involve creating a test database and running your service code using more realistic scenarios.

In order to achieve that you must construct your own `DatabaseManager` instance and replace the desired database connection with the required
driver (for example SQLite):

```php
use Cycle\ORM;
use Cycle\Database;

$dbal = new Database\DatabaseManager(new Database\Config\DatabaseConfig([
    'default'     => 'default',
    'databases'   => [
        'default' => ['connection' => 'sqlite']
    ],
    'connections' => [
        'sqlite' => [
            'driver'  => Database\Driver\SQLite\SQLiteDriver::class,
            'connection' => 'sqlite::memory:',
            'username'   => '',
            'password'   => '',
        ]
    ]
]));

$orm = new ORM\ORM(new ORM\Factory($dbal));

// you can use already calculated database schema
$orm = $orm->withSchema(new ORM\Schema($cachedSchema));
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
