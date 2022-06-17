# Relation collections

Collection factory is responsible for creation and filling `*Many` relation collections.

> Cycle ORM used to use `doctrine/collections` as a default collection for `*Many` relations,
> but since v2.x it doesn't use `doctrine/collections` out of the box anymore.
> Now you have an ability to opt which collection type will be used for `*Many` relations by default and which for
> specific entities relations.

## Collection factories

- `Cycle\ORM\Collection\ArrayCollectionFactory` - default collection factory, uses PHP arrays as collection.
- `Cycle\ORM\Collection\DoctrineCollectionFactory` - uses `doctrine/collections` package (should be installed manually)
- `Cycle\ORM\Collection\IlluminateCollectionFactory` - uses `illuminate/collection` package (should be installed
  manually)
- `Cycle\ORM\Tests\Unit\Collection\LoophpCollectionFactoryTest` - uses `loophp\collection` package (should be installed
  manually)

### Configuration

By default, Cycle ORM uses `Cycle\ORM\Collection\ArrayCollectionFactory`.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class)]
    public array $posts;
}
```

In order to use alternate collection type by default, you need to pass desired collection factory as a second
argument (`defaultCollectionFactory`) to `Cycle\ORM\Factory` object:

```php
use Cycle\ORM;

$schema = new ORM\Schema(...);

$factory = (new ORM\Factory(
    dbal: $dbal,
    defaultCollectionFactory: new ORM\Collection\ArrayCollectionFactory    // Default collection factory
))
    // requires doctrine/collections package
    ->withCollectionFactory(
        'doctrine',                                         // Alias
         new ORM\Collection\DoctrineCollectionFactory,
         \Doctrine\Common\Collections\Collection::class    // <= Base collection
    )
    
    // requires illuminate/collections package
    ->withCollectionFactory(
        'illuminate', 
        new ORM\Collection\IlluminateCollectionFactory, 
        \Illuminate\Support\Collection::class
    );

$orm = new ORM\ORM(
    factory: $factory,
    schema: $schema
);
```

> Method `Factory::withCollectionFactory` returns a new, immutable Factory object, and you need to rebind factory
> for `Cycle\ORM\ORM` object after adding a new collection factory.

```php
use Cycle\ORM;

$orm = new ORM\ORM(...);

$container = new Container();
$container->bindSingleton(ORM\ORMInterface::class, $orm);

$factory = $orm->getFactory()
    ->withCollectionFactory(
        'doctrine',
         new ORM\Collection\DoctrineCollectionFactory,
          \Doctrine\Common\Collections\Collection::class
    );

$orm = $orm->with(factory: $factory);

$container->bindSingleton(ORM\ORMInterface::class, $orm);
```

### Relation collection type definition

#### Using entity schema

```php
class CommentCollection extends \Doctrine\Common\Collections\ArrayCollection {
    public function filterActive(): self { /* ... */ }
    public function filterHidden(): self { /* ... */ }
}

$schema = [
    User::class => [
        //...
        Schema::RELATIONS   => [
            'posts' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Post::class,
                Relation::COLLECTION_TYPE => null, // <= Will be used a default collection factory
                Relation::SCHEMA => [ /*...*/ ],
            ],
            'comments' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Comment::class,
                Relation::COLLECTION_TYPE => 'doctrine', // <= Will be used collection factory with alias doctrine
                Relation::SCHEMA => [ /*...*/ ],
            ],
            'tokens' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Token::class,
                Relation::COLLECTION_TYPE => \Doctrine\Common\Collections\Collection::class, // <= Will be used collection factory with matching by base class
                Relation::SCHEMA => [ /*...*/ ],
            ]
        ]
    ],
    Post::class => [
        //...
        Schema::RELATIONS   => [
            'comments' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Comment::class,
                Relation::COLLECTION_TYPE => CommentsCollection::class, // <= Will be used collection factory with matching by base class
                Relation::SCHEMA => [ /*...*/ ],
            ]
        ]
    ]
];
```

#### Using entity annotation

```php
use Doctrine\Common\Collections\ArrayCollection as DoctrineCollection;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasOne;
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\Annotated\Annotation\Relation\ManyToMany;
use loophp\collection\Collection as LoophpCollection;

#[Entity]
class User
{
    #[Column(type: "primary")]
    protected int $id;
    
    #[HasMany(target: Friend::class)]
    protected array $friends = [];
    
    #[HasMany(target: Post::class, collection: LoophpCollection::class)]
    protected LoophpCollection $posts = [];
    
    #[HasMany(target: Profile::class, collection: 'doctrine')]
    protected DoctrineCollection $posts;
   
    #[ManyToMany(target: Tag::class, load: through=TagMap::class, collection: CommentsCollection::class)]
    protected CommentsCollection $tags;
}
```

## Accessing Collection

The ORM will automatically instantiate a collection instance for your relations, however, you are still required to
initiate empty collections in your constructor to use newly created entities:

```php
use Doctrine\Common\Collections\ArrayCollection;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class, collection: 'doctrine')]
    public ArrayCollection $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }
}
```

The collection property will be set automatically on the selection:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->with('posts')->limit(1)->fetchOne();

print_r($user->posts);
```

## Collection API

You can create your onw collection factories by implementing `Cycle\ORM\Collection\CollectionFactoryInterface` interface

```php
use Cycle\ORM\Collection\CollectionFactoryInterface

class ArrayCollectionFactory implements CollectionFactoryInterface
{
    public function withCollectionClass(string $class): static
    {
        // Do nothing
        return $this;
    }

    public function collect(iterable $data): array
    {
        return match (true) {
            \is_array($data) => $data,
            $data instanceof \Traversable => \iterator_to_array($data),
            default => throw new CollectionFactoryException('Unsupported iterable type.'),
        };
    }
}
```
