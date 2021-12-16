# Event listener

Events serve as a great way to decouple various aspects of your application, since a single event can have multiple
listeners that do not depend on each other.

This package provides a simple observer pattern implementation, allowing you to subscribe and listen for various events.

## Usage

If you are listening for many events on a given entity, you may use `Cycle\ORM\Entity\Behavior\EventListener` attribute
to group all of your listeners into a single class. Event listener classes should have attributes which subscribe to the
Entity events you wish to listen for. Each of these methods receives the event object as their only argument.

### Entity event listener

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior;

#[Entity]
#[Behavior\EventListener(
    listener: CommentListener::class
)]
class Comment
{
    #[Column(type: 'primary')]
    public int $id;
    
    #[Column(type: 'int')]
    public ?int $parentId = null;
    
    #[Column(type: 'string')]
    public int $body;
}
```

### Listener class

Next, let's take a look at the listener for our example Comment entity. Event listener receive event instances in their
methods that subscribed to the events.

```php
use Cycle\ORM\Entity\Behavior\Attribute\Listen;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command;
use Cycle\ORM\EntityManagerInterface;

final class CommentListener
{
    public function __construct(
        private CommentSpamFilter $spamFilter,
        private CommentRepository $commentRepository,
        private EntityManagerInterface $em
    ) {
    }
    
    #[Listen(Command\OnCreate::class)]
    #[Listen(Command\OnUpdate::class)]
    public function filterSpam(Command\OnCreate|Command\OnUpdate $event): void
    {
        $event->state->register(
            'body', 
            $this->spamFilter->filter(
                $event->state->getData()['body']
            )
        );
    }
    
    #[Listen(Command\OnDelete::class)]
    public function deleteChildComments(Command\OnDelete $event): void
    {
        // Please don't use this example in production.
        // This example contains recursion with too low performance.
        $comments = $this->commentRepository->findAll([
           'parent_id' => $event->entity->id
        ]);
        
        foreach ($comments as $comment) {
            $this->em->delete($comment);
        }
        
        // Will fire 'Command\OnDelete' for each child comment
        $this->em->run(); 
    }
}
```

In this example, the `CommentListener` needs to spam filter, repository and entity manager services. In this context,
all dependencies will automatically be resolved and injected into the class from
your [application container](/docs/en/entity-behaviors/install.md).

## Available events:

- [OnCreate](/docs/en/entity-behaviors/events.md#oncreate)
- [AfterCreate](/docs/en/entity-behaviors/events.md#aftercreate)
- [OnUpdate](/docs/en/entity-behaviors/events.md#onupdate)
- [AfterUpdate](/docs/en/entity-behaviors/events.md#afterupdate)
- [OnDelete](/docs/en/entity-behaviors/events.md#ondelete)
- [AfterDelete](/docs/en/entity-behaviors/events.md#afterdelete)

