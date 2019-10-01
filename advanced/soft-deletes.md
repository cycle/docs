# Soft Deleted Entities
The soft deletion functionality can be achieved by applying custom delete strategy in mapper and global entity constrain to limit the selection.

## Mapper
You can alter `queueDelete` method of the mapper and replace it with `Update` command instead:

```php
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
        // this sequence is only required if entity created and deleted within
        // one transaction
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
public function queueDelete($entity, Node $node, State $state): CommandInterface
{
    if ($state->getStatus() == Node::SCHEDULED_DELETE) {
        return parent::queueDelete($entity, $node, $state);
    }

    // ...
}
```

Example usage:

```php
$orm->getHeap()->get($user)->setStatus(Node::SCHEDULED_DELETE);

$tr = new Transaction($orm);
$tr->delete($user);
$tr->run();
```

## Constrain
To filter only deleted entities create the constraint:

```php
class NotDeletedConstrain implements ConstrainInterface
{
    public function apply(QueryBuilder $query)
    {
        $query->where('deleted_at', '=', null);
    }
}
```

## Usage
To enable soft deletes for your entity associate newly created mapper and constrain with it:

```php
/** @Entity(mapper="SoftDeletedMapper", constrain="NotDeletedConstrain") */
class User
{
    // ...
}
```

Now all entity deletes will issue Update command instead.

## Select Deleted
You can select deleted entities from the database by disabling your select constrain:

```php
$userSelect = $orm->getRepository(User::class)->select();
$userSelect->constrain(null);
```
