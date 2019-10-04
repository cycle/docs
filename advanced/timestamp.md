# CreatedAt/UpdatedAt Timestamps
It is possible to automatically set `created_at` and `updated_at` column values on entity update. In order to achieve that you have to write a
custom mapper implementation to automatically register these values in entity commands.

## Timestamped Mapper
The simplest mapper will look like:

```php
class TimestampedMapper extends Mapper
{
    public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface
    {
        $cmd = parent::queueCreate($entity, $node, $state);

        $state->register('created_at', new \DateTimeImmutable(), true);
        $cmd->register('created_at', new \DateTimeImmutable(), true);

        $state->register('updated_at', new \DateTimeImmutable(), true);
        $cmd->register('updated_at', new \DateTimeImmutable(), true);

        return $cmd;
    }
    public function queueUpdate($entity, Node $node, State $state): ContextCarrierInterface
    {
        /** @var Update $cmd */
        $cmd = parent::queueUpdate($entity, $node, $state);

        $state->register('updated_at', new \DateTimeImmutable(), true);
        $cmd->registerAppendix('updated_at', new \DateTimeImmutable());

        return $cmd;
    }
}
```

> While we can set column values directly in the `Insert` command, we have to use the alternative method `registerAppendix` for `Update`. This method will only push changes to the database if any other entity field has changes (for example if entity FK has been updated through the relation).

## Automatically Define Columns
You can use the annotated entities extension to automatically declare the needed columns from inside your mapper:

```php
/**
 * @Table(
 *      columns={"created_at": @Column(type="datetime"), "updated_at": @Column(type"=datetime")},
 * )
 */
class TimestampedMapper extends Mapper
{
    public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface
    {
        $cmd = parent::queueCreate($entity, $node, $state);

        $state->register('created_at', new \DateTimeImmutable(), true);
        $cmd->register('created_at', new \DateTimeImmutable(), true);

        $state->register('updated_at', new \DateTimeImmutable(), true);
        $cmd->register('updated_at', new \DateTimeImmutable(), true);

        return $cmd;
    }
    public function queueUpdate($entity, Node $node, State $state): ContextCarrierInterface
    {
        /** @var Update $cmd */
        $cmd = parent::queueUpdate($entity, $node, $state);

        $state->register('updated_at', new \DateTimeImmutable(), true);
        $cmd->registerAppendix('updated_at', new \DateTimeImmutable());

        return $cmd;
    }
}
```

Associate your entity with the newly created mapper in order to register these columns on schema update:

```php
/** @Entity(mapper="TimestampedMapper") */
class User
{
    // ...
}
```

> You can use one mapper for multiple entities.
