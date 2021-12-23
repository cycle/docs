# Soft Deleted Entities
The soft deletion functionality can be achieved by applying a custom delete strategy in the mapper, combined with a global entity scope to limit the selection.

## Mapper
You can alter the `queueDelete` method of the mapper and replace it with an `Update` command instead:

```php
use Cycle\ORM\Mapper\Mapper;
use Cycle\ORM\Heap\Node;
use Cycle\ORM\Heap\State;
use Cycle\ORM\Command\CommandInterface;
use Cycle\ORM\Context\ConsumerInterface;
use Cycle\ORM\Command\Database\Update;

class SoftDeletedMapper extends Mapper
{
    public function queueDelete($entity, Node $node, State $state): CommandInterface
    {
        // identify entity as being "deleted"
        $state->setStatus(Node::SCHEDULED_DELETE);
        $state->decClaim();

        $cmd = new Update(
            $this->source->getDatabase(),
            $this->source->getTable(),
            ['deleted_at' => new \DateTimeImmutable()]
        );

        // forward primaryKey value from entity state
        // this sequence is only required if the entity is created and deleted 
        // within one transaction
        $cmd->waitScope($this->primaryColumn);
        $state->forward(
            $this->primaryKey,
            $cmd,
            $this->primaryColumn,
            true,
            ConsumerInterface::SCOPE
        );

        return $cmd;
    }
}
```

You can permanently delete needed entities by using DBAL directly or by adding a switch to the mapper to fallback to the original command.

```php
use Cycle\ORM\Heap;

// ...

public function queueDelete($entity, Heap\Node $node, Heap\State $state): \Cycle\ORM\Command\CommandInterface
{
    if ($state->getStatus() == Heap\Node::SCHEDULED_DELETE) {
        return parent::queueDelete($entity, $node, $state);
    }

    // ...
}
```

Example usage:

```php
$orm->getHeap()->get($user)->setStatus(\Cycle\ORM\Heap\Node::SCHEDULED_DELETE);

$tr = new \Cycle\ORM\Transaction($orm);
$tr->delete($user);
$tr->run();
```

## Scope
To filter out deleted entities create the scope:

```php
use Cycle\ORM\Select;

class NotDeletedScope implements Select\ScopeInterface
{
    public function apply(Select\QueryBuilder $query)
    {
        $query->where('deleted_at', '=', null);
    }
}
```

## Usage
To enable soft deletes for your entity associate the newly created mapper and scope with it:

```php
/** @Entity(mapper="SoftDeletedMapper", scope="NotDeletedScope") */
class User
{
    // ...
}
```

Now all entity deletes will issue Update commands instead.

## Select Deleted
You can select deleted entities from the database by disabling your select scope:

```php
$userSelect = $orm->getRepository(User::class)->select();
$userSelect->scope(null);
```
