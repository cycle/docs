# Commands and Linked Contexts
All of the persist operations are performed using a set of linked commands. Commands responsible for execution, rollback and change commitment. Command can depend on values provided by entity state or another command (link).

## Command Interface
To understand how commands work let's review the underlying interface first:

```php
interface CommandInterface
{
    /**
     * Must return true when command is ready for the execution. UnitOfWork will throw
     * an exception if any of the command will stuck in non ready state.
     *
     * @return bool
     */
    public function isReady(): bool;
    
    /**
     * Indicates that command has been executed.
     *
     * @return bool
     */
    public function isExecuted(): bool;
    
    /**
     * Executes command.
     */
    public function execute();
    
    /**
     * Complete command, method to be called when all other commands are already executed and
     * transaction is closed.
     */
    public function complete();
    
    /**
     * Rollback command or declare that command been rolled back.
     */
    public function rollBack();
}
```

One of the most important methods in commands is `isReady`, this method used by Transaction to properly sort dependency graph in order
to execute commands in the most optimal order.

## Linked Commands
We can review a simple command setup in which one command declared column which depends on lastInsertID provided by another command:

```php
use Cycle\ORM\Command\Database;

$update = new Database\Update($dbal->database('default'), 'table', $data, $where);
$insert = new Insert($dbal->database('default'), 'table', $data);
```

Now we have to declare for update command to wait for the context provided by insert:

```php
// command would be not ready until the context is provided
$update->waitContext('some_id');
```

And ask Insert command to forward context value once it's available:

```php
$insert->forward(Insert::INSERT_ID, $update, 'some_id');
```

Now, no matter what in which order commands were added to the Transaction the Update will always be executed after Insert command.

## Joining Commands in Mappers
If you want to implement and issue custom commands you must pick a place where to create them. You can do inside custom relation `queue` method or alter one of methods of entity mappers.

Since you can only issue one command from your mapper you can use `Sequence` or `ContextSequence` to merge commands together:

> CommandSequence automatically forwards link requests to the primary command.

```php
public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface
{
    $cc = parent::queueCreate($entity, $node, $state);
    
    $cs = new ContextSequence();
    $cs->addPrimary($cc);
    $cs->addCommand(new CustomCommand());
    
    return $cs;
}
```

## Command Topology
Besides sequences, you also have multiple system commands which can be used to create more complex execution trees.

#### Nil Command
You can use Nil command to state that no changes must be made:

```php
use Cycle\ORM\Command\Branch;

// ...

public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface
{
    // disable create
    return new Branch\Nil();
}
```

#### Condition Command
In some cases you might want to execute command or branch of commands using some external condition, you can use `Condition` command for this purpose:

```php
use Cycle\ORM\Command\Branch;

// ...

public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface
{
    $cc = parent::queueCreate($entity, $node, $state);
    
    return new Branch\Condition($cc, function() {
        return mt_rand(1, 0) === 1; // randomly drop some commands, don't do it.
    });
}
```

You can link this conditions to the entity state or node to implement more complex logic.

> Make sure to use entity State not Node as condition variable as State will change during the exeuction while Node would not.

## Split command
Another internal command which is applied by ORM by default is `Split`. The command is used in order to resolve cyclic dependencies by splitting the persistence of the object into Insert and Update.
