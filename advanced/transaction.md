# Transactions (Unit of Work)
Every persist operation with entity must be performed via Transaction object. This object will rely on `Heap` associated with given ORM
and will request each entity mapper and relation to issue set of persist commands in a form of command chain.

## TransactionInterface
ORM provides convinient class to manage transactions, however, it is recommended to couple your code with underlying interface for 
the simplicity and easier testing going forward:

```php
interface TransactionInterface
{
    // how to store/delete entity
    public const MODE_CASCADE     = 0;
    public const MODE_ENTITY_ONLY = 1;
    
    /**
     * Persist the entity.
     *
     * @param object $entity
     * @param int    $mode
     */
    public function persist($entity, int $mode = self::MODE_CASCADE);
    
    /**
     * Delete entity from the database.
     *
     * @param object $entity
     * @param int    $mode
     */
    public function delete($entity, int $mode = self::MODE_CASCADE);
    
    /**
     * Execute all nested commands in transaction, if failed - transaction MUST automatically
     * rollback and exception instance MUST be thrown.
     *
     * @throws \Throwable
     */
    public function run();
}
```

## Example Usage
To persist your entity simply add it to transaction using `persist` method and call method `run` after that.

```php
$t = new Transaction($orm);
$t->persist($e);
$t->run();
```

> Do not forget to handle `Spiral\Database\Exception\DatabaseException`.

You can handle multiple transactions within one scope, they can overlap or be responsible for separate entities:

```php
$t = new Transaction($orm);
$t->persist($e);
$t->run();

$t2 = new Transaction($orm);
$t2->persist($e2);
$t2->run();
```

After execution Transaction will be cleared and can be re-used for next batch of commands, please note, ORM Heap would not be reset 
after the transaction, you must clean it manually if needed.

```php
$t = new Transaction($orm);

for ($i=0; $i<1000; $i++) {
    $t->persist($e);
    $t->run();
}

$orm->getHeap()->clean();
```

## Cascade persisting
By default, Transaction will create command chain to store all entity relations unless the opposite is specified in relation schema.
If you would like to store only entity content without it's relations use persist option `MODE_ENTITY_ONLY`:

```php
$t = new Transaction($orm);
$t->persist($e, Transaction::MODE_ENTITY_ONLY);
$t->run();
```

## Transaction Runner
In some cases you might want to implement your own persist logic or sorting for issued commands. You can create your own
Transaction implementation or provide second argument to the transaction to handle actual command execution:

```php
interface RunnerInterface extends \Countable
{
    /**
     * @param CommandInterface $command
     */
    public function run(CommandInterface $command);
    
    /**
     * Complete/commit all executed changes. Must clean the state of the runner.
     */
    public function complete();
    
    /**
     * Rollback all executed changes. Must clean the state of the runner.
     */
    public function rollback();
}
```

For example we can log the order of which commands are being executed by wrapping original transaction Runner:

```php
class LogRunner implements RunnerInterface
{
    private $runner;
    
    public function __construct()
    {
        $this->runner = new Cycle\ORM\Transaction\Runner();
    }
  
    public function run(CommandInterface $command)
    {
        print_r($command);
        $this->runner->run($command);
    }
    
    public function complete()
    {
        $this->runner->complete();
    }
    
    public function rollback()
    {
        $this->runner->rollback();
    
    }
}
```

To use it:

```php
$t = new Transaction($orm, new LogRunner());
$t->persist($e);
$t->run();
```
