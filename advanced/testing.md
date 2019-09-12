# Testing
Cycle ORM attempts to simplify the testing of your application by providing well isolated interfaces to load and persist your entities.

## Classic Mock Approach
First approach would involve mocking instances of entity repositories and `TransactionInterface`, for example given code can be well tested without ORM initization:

```php
public function addOrder(User $u, Order $o, TransactionInterface $t)
{
    $u->orders->add($o);
    $t->persist($u)->run();
}
```

Mock `TransactionInterface` to check the state of the entity after method call. Additionally you can mock custom entity repositories:

```php
class MyService
{
    private $users;
    
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function disableUser($id, TransactionInterface $t)
    {
        $u = $this->users->findByPK($id);
        $u->status = 'disabled';
        $t->persist($u)->run();
    }
}
```

> You can also `run` transaction outside of your service method if you wish to group multiple persist operations together.

## Mocking The Database
Another, more complex approch would invole creating test database and running your service code using more realistic scenarios.

In order to achieve that you must construct your own `DatabaseManager` instance and replace desired database connection with needed
driver (for example SQLite):

```php
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

$orm = new ORM(new Factory($dbal));

// you can use already calculated database schema
$orm = $orm->withSchema(new Schema($cachedSchema));
```

You can clean your database state after each iteration using schema introspection appoach:

```php
public function tearDown()
{
    $db = $this->dbal->database('default');
    
    // delete all FKs first
    foreach ($database->getTables() as $table) {
        $schema = $table->getSchema();
        foreach ($schema->getForeignKeys() as $foreign) {
            $schema->dropForeignKey($foreign->getColumn());
        }
        
        $schema->save(Handler::DROP_FOREIGN_KEYS);
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
