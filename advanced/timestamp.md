# Timestamped Entities
It is possible to automatically set `created_at`, `deleted_at` column values on entity update. In order to achieve that you have to write
custom mapper implementation which will be automatically registering such values in entity commands.

## Timestamped Mapper
Simples mapper will look like:

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

> While we can set column values directly in `Insert` command we have to use alternative method `registerAppendix` for `Update`. Such method will only push changes to database if any other entity field has changes.

## Automatically Define Columns
You can use annotated entites extension to automatically declared needed column from inside your mapper:

```php
/**
 * @Table(
 *      columns={created_at: @Column(type=datetime), updated_at: @Column(type=datetime)},
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

Associate your entity with newly created mapper in order to register this columns on schema update:

```php
/** @Entity(mapper="TimestampedMapper") */
class User
{
    // ...
}
```

> You can use one mapper for multiple entities.
