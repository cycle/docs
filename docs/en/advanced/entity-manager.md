# Entity manager (Unit of Work)

Every persist operation for an entity must be performed via the Entity manager object. This object will rely on `Heap`
associated with the given ORM and will request each entity mapper and relation to issue a set of persist commands in the
form of a command chain.

> All transactions are treated as disposable, you can create and delete them as you need.

## EntityManagerInterface

ORM provides a convenient class to manage transactions, however, it is recommended to couple your code with an
underlying interface for the simplicity and easier testing going forward:

[//]: # (TODO добавить интерфейс)

```php
interface EntityManagerInterface
{
    
}
```

## Example Usage

To persist your entity simply add it to the entity manager using the `persist` method and call the method `run` after
that.

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($entity);
$manager->run();
```

> Do not forget to handle `Cycle\Database\Exception\DatabaseException`.

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($entity);
try {
    $manager->run();
} catch (\Cycle\Database\Exception\StatementException\ConnectionException $e) {
    $logger->error($e->getMessage());
}
```

Or you can return transaction state and handle errors via it:

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($entity);
$state = $manager->run(throwException: false);

while ($error = $state->getLastError()) {
    if ($error instanceof \Cycle\Database\Exception\StatementException\ConnectionException) {
        usleep(100);
        $state->retry();
        continue;
    }
    
    throw $error;
}
```

You can handle multiple entity managers within one scope. They can overlap or be responsible for separate entities:

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($e);
$manager->run();

$manager2 = new \Cycle\ORM\EntityManager($orm);
$manager2->persist($e2);
$manager2->run();
```

After execution the Entity manager will be cleared and can be re-used for the next batch of commands. Please note, ORM
Heap would not be reset after the transaction, you must clean it manually if needed.

```php
$manager = new \Cycle\ORM\EntityManager($orm);

for ($i=0; $i<1000; $i++) {
    $manager->persist($e);
    $manager->run();
}

$orm->getHeap()->clean();
// or

$manager->clean(cleanHeap: true);
```

## Cascade persisting

By default, EntityManager will create a command chain to store all entity relations unless the opposite is specified in
the relation schema. If you would like to store only entity content without it's relations use the persist named
argument `cascade`:

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($entity, cascade: false);
$manager->run();
```

## Entity manager runner

In some cases, you might want to implement your own persist logic or sorting for issued commands. You can create your
own Entity manager implementation or provide the second argument to the transaction to handle actual command execution:

```php
use Cycle\ORM\Command\CommandInterface;

interface RunnerInterface extends \Countable
{
    public function run(CommandInterface $command): void;

    /**
     * Complete/commit all executed changes. Must clean the state of the runner.
     */
    public function complete(): void;

    /**
     * Rollback all executed changes. Must clean the state of the runner.
     */
    public function rollback(): void;
}
```

For example, we can log the order of which commands are being executed by wrapping the original transaction Runner:

```php
use Cycle\ORM\Command\CommandInterface;

class LogRunner implements RunnerInterface
{
    private RunnerInterface $runner;

    public function __construct(
        private LoggerInterface $logger
    ) {
        $this->runner = new \Cycle\ORM\Transaction\Runner();
    }

    public function run(CommandInterface $command): void
    {
        $this->logger->debug($command);
        
        $this->runner->run($command);
    }

    public function complete(): void
    {
        $this->runner->complete();
    }

    public function rollback(): void
    {
        $this->runner->rollback();
    }
}
```

To use it:

```php
$manager = new \Cycle\ORM\EntityManager($orm, new LogRunner());
$manager->persist($entity);
$manager->run();
```

## Reusing same Entity manager

The Entity manager is clean after the `run` invocation. You must assemble a new transaction to retry the same set of
entities. Since the Entity manager is clean you are able to reuse it over and over again (make sure to keep `persist`
, `delete` and `run` operations within one method).

## Entity manager transactions level

Method `run` wraps all commands into one Database transaction and tries to commit it. In some cases there is a need not
use a new transaction level and run commands inside current transaction.

```php
$driver->beginTransaction();

$manager = new \Cycle\ORM\EntityManager($orm);

try {
    $driver->execute('update users set name = "John Smith" where id = 1');
    
    $manager->persist($entity);
    $manager->persist($entity2);
    $manager->delete($entity3);
    
    $manager->run(runner: \Cycle\ORM\Transaction\Runner::continueTransaction());

    $driver->commitTransaction();
} catch (\Throwable) {
    $driver->rollbackTransaction();
}
```
