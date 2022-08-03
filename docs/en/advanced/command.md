# Commands and Linked Contexts

All the persist operations are performed using a set of linked commands. Commands are responsible for execution,
rollback and change commitment. A command can depend on values provided by entity state or another command (link).

## Command Interface

[//]: # (TODO интерфейс изменился, необъодимо переделать)

To understand how commands work let's review the underlying interface first:

```php
use Cycle\Database\DatabaseInterface;

interface CommandInterface
{
    /**
     * Must return true when command is ready for the execution. UnitOfWork will throw
     * an exception if any of the command will stuck in non ready state.
     */
    public function isReady(): bool;

    /**
     * Indicates that command has been executed.
     */
    public function isExecuted(): bool;

    /**
     * Executes command.
     */
    public function execute(): void;

    public function getDatabase(): ?DatabaseInterface;
}
```

One of the most important methods in commands is `isReady`. This method is used by Transaction to properly sort the
dependency graph in order to execute commands in the most optimal order.

> **Note**
> The page is in progress
