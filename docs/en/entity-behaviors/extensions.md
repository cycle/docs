# Custom behaviors (Extension)

Custom behaviors are the primary way of adding functionality for entities. Behaviors might be anything from a great way
to work with dates like Carbon or a behavior that allows you to create slug from Post title property. You have the
ability to create your own behaviors and share them with the community.

Custom behaviors are classes that allow implementing custom logic for your entities. They extend
`Cycle\ORM\Entity\Behavior\Schema\BaseModifier` and implement the custom logic in abstract methods.

## Example

In the following example we will show you how it is simple to create a custom behavior:

### Attribute

At first, you have to describe the attribute that will be used in an entity.

```php
use Cycle\ORM\Entity\Behavior\Schema\BaseModifier;
use Cycle\ORM\Entity\Behavior\Schema\RegistryModifier;
use Cycle\Schema\Registry;

#[\Attribute(\Attribute::TARGET_CLASS)]
final class Sluggable extends BaseModifier
{
    public function __construct(
        // Properties should present in your entity with `[#Column(type: 'string')]` attribute
        private string $field = 'slug',
        private string $from = 'title'
    ) {
    }

    protected function getListenerClass(): string
    {
        return SluggableListener::class;
    }

    protected function getListenerArgs(): array
    {
        // Will be passed to the listener constructor
        return [
            'field' => $this->field,
            'from' => $this->from
        ];
    }
}
```

### Attribute listener

Then you have to describe listener connected with `Sluggable` attribute via `Sluggable::getListenerClass` method.

```php
use Cycle\ORM\Entity\Behavior\Attribute\Listen;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command\OnCreate;
use Cycle\ORM\Entity\Behavior\Event\Mapper\Command\OnUpdate;
use Cocur\Slugify\Slugify;

final class SluggableListener
{
    public function construct(
        private Slugify $slugify,
        private ORMInterface $orm,
        private string $field
        private string $from
    ) {
    }

    #[Listen(OnCreate::class)]
    #[Listen(OnUpdate::class)]
    public function invoke(OnCreate|OnUpdate $event): void
    {
        $slug = $this->slugify->slugify($event->state->getData()[$this->from]);
        $isExist = $this->orm->getRepository($event->role)->findOne([$this->field => $slug]) !== null;

        if ($isExist) {
            $slug .= '-' . time();
        }

        $event->state->register($this->field, $slug);
    }
}
```

### Available events:

- [OnCreate](/docs/en/entity-behaviors/events.md#oncreate)
- [AfterCreate](/docs/en/entity-behaviors/events.md#aftercreate)
- [OnUpdate](/docs/en/entity-behaviors/events.md#onupdate)
- [AfterUpdate](/docs/en/entity-behaviors/events.md#afterupdate)
- [OnDelete](/docs/en/entity-behaviors/events.md#ondelete)
- [AfterDelete](/docs/en/entity-behaviors/events.md#afterdelete)


### Example of usage

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
#[Sluggable(
    field: 'slug',
    from: 'title'
)]
class Comment
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $title;

    #[Column(type: 'string')]
    private string $slug = '';
}
```
