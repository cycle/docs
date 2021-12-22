# Hooks (Closure based event listeners)

Events serve as a great way to decouple various aspects of your application, since a single event can have multiple
listeners that do not depend on each other.

This package provides a simple observer pattern implementation, allowing you to subscribe and listen for various events
via closure based event listeners.

## Usage

To start listening to entity events, add a `Cycle\ORM\Entity\Behavior\Hook` attribute to your Entity.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;
use Cycle\ORM\Entity\Behavior;

#[Entity]
#[Behavior\Hook(
    callable: [Comment::class, 'onCreate'], 
    events: Command\OnCreate::class
)]
#[Behavior\Hook(
    callable: [Comment::class, 'AfterCreate'], 
    events: Command\AfterCreate::class
)]
class Comment
{
    #[Column(type: 'primary')]
    public int $id;
    
    public static function onCreate(Command\OnCreate $event): void
    {
        // do something before comment created
    }
    
    public static function afterCreate(Command\AfterCreate $event): void
    {
        // do something when comment created
    }
}
```

You can subscribe on one or many events.

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;
use Cycle\ORM\Entity\Behavior;

#[Entity]
#[Behavior\Hook(
    callable: [Comment::class, 'events'], 
    events: [Command\OnCreate::class, Command\AfterCreate::class]
)]
class Comment
{
    #[Column(type: 'primary')]
    public int $id;
    
    public static function events(Command\OnCreate|Command\AfterCreate $event): void
    {
        if ($event instanceof Command\OnCreate) {
            // do something before comment created
        }
        
        if ($event instanceof Command\AfterCreate) {
            // do something when comment created
        }
    }
}
```

## Available events:

- [OnCreate](/docs/en/entity-behaviors/events.md#oncreate)
- [AfterCreate](/docs/en/entity-behaviors/events.md#aftercreate)
- [OnUpdate](/docs/en/entity-behaviors/events.md#onupdate)
- [AfterUpdate](/docs/en/entity-behaviors/events.md#afterupdate)
- [OnDelete](/docs/en/entity-behaviors/events.md#ondelete)
- [AfterDelete](/docs/en/entity-behaviors/events.md#afterdelete)
