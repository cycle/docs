# References

The ORM provides the ability to reference to objects that are not currently loaded from the database. This can be
achieved using main abstraction: **Reference**.

References provide the ability to replace an actual entity with a Reference object which points to the entity's role and
it's scope (columns used for relations). In some cases, like in relations `belongsTo` and `refersTo`, the object would
not be fetched from the database and the manually specified scope will be used.

We can demonstrate it:

```php
#[Entity]
class Post
{
    #[Column(type: 'primary')]
    private int $id;

    #[BelongsTo(target: User::class)]
    private $user;
}
```

Now, if we want to create a new `Post` entity we have an option to set the user value as a `User` entity or use a
reference instead:

```php
$post = new Post();
$post->user = new \Cycle\ORM\Reference\Reference('user', ['id' => 1]);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post)->run();
```

> You must use entity role in reference.

If you use references often it might be convenient to create custom object for such purpose:

```php
use Cycle\ORM\Reference\ReferenceInterface;

class UserID implements ReferenceInterface
{
    public function __construct(
        private int $id
    ) {
    }

    public function getRole(): string
    {
        return 'user';
    }

    public function getScope(): array
    {
        return ['id' => $this->id];
    }
}
```

And use it accordingly:

```php
$post = new Post();
$post->user = new UserID(1);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($post)->run();
```
