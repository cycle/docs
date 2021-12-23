# Events

The package dispatch several events, allowing you to hook into the following moments in an entity's lifecycle:

An event object contains the following public properties:

```php
// Entity role
print_r($event->role);      

// Entity mapper
print_r($event->mapper);     

// Read only entity object. Change entity property values carefully.
// Property value changes won't affect persisting data, but will affect next event listeners.
print_r($event->entity);    

// Use for changing persisting data. Data from state will be stored to the database
print_r($event->state);

// DateTime object shared between all events
print_r($event->timestamp); 

// Entity state before changes
print_r($event->node); 
```

## OnCreate

The event will dispatch before an entity is stored to the database.

```php

use Cycle\ORM\Entity\Behavior\Attribute\Listen;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;

#[Listen(OnCreate::class)]
public function onCreate(Command\OnCreate $event): void
{
    $event->state->register('id', Uuid::uuid4());
}
```

## OnUpdate

The event will dispatch before an entity is updated in the database.

```php
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;

public function onUpdate(Command\OnUpdate $event): void
{
    $event->state->register('updated_at', new \DateTimeImmutable());
}
```

[//]: # (## AfterUpdate)

[//]: # (The event will dispatch after an entity is updated in the database.)

[//]: # (```php)

[//]: # (use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;)

[//]: # (public function afterUpdate&#40;Command\AfterUpdate $event&#41;: void)

[//]: # ({)

[//]: # (    if &#40;!$event->entity instanceof LoggableInterface&#41; {)

[//]: # (        return;)

[//]: # (    })

[//]: # (    )
[//]: # (    $this->logRepository->store&#40;[)

[//]: # (        'entity_role' => $event->role,)

[//]: # (        'entity_id' => $event->entity->getId&#40;&#41;,)

[//]: # (        'created_at' => new \DateTimeImmutable&#40;&#41;,)

[//]: # (        'changed_by' => $event->entity->getAuthor&#40;&#41;->getId&#40;&#41;,)

[//]: # (        'changes' => $event->state->getChanges&#40;&#41;,)

[//]: # (    ]&#41;;)

[//]: # (})

[//]: # (```)

## OnDelete

The event will dispatch before an entity is deleted from the database.

```php
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;

public function onDelete(Command\OnDelete $event): void
{
    if (!$event->entity->getAuthor()->hasRole('admin')) {
        throw new UnauthorizedException('You don\'t have right to delete this entity.');
    }
}
```

[//]: # (## AfterDelete)

[//]: # (The event will dispatch after an entity is deleted from the database.)

[//]: # (```php)

[//]: # (use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;)

[//]: # (public function afterDelete&#40;Command\AfterDelete $event&#41;: void)

[//]: # ({)

[//]: # (    $this->fileStorage->deleteAttachements&#40;)

[//]: # (        $event->role, )

[//]: # (        $event->entity->getId&#40;&#41;)

[//]: # (    &#41;;)

[//]: # (})

[//]: # (```)

## QueueCommand

The event will dispatch all the events for an entity.

```php
use Cycle\ORM\Entity\Behavior\Attribute\Listen;
use Cycle\ORM\Entity\Behavior\Event\Mapper\QueueCommand;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;

#[Listen(QueueCommand::class)]
public function onCreate(QueueCommand $event): void
{
    if ($event instanceof Command\OnCreate){
        // ...
    }
    
    if ($event instanceof Command\OnUpdate){
        // ...
    }
    
    if ($event instanceof Command\OnDelete){
        // ...
    }
}
```
