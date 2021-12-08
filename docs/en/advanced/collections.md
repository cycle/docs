# Collections

> В CycleORM v2 была убрана жесткая привязка к определенному типу коллекции.
> Начиная с этой версии вы можете выбирать какой тип коллекции будет использоваться для хранения данных связей для всех сущностей по умолчанию и для конкретной сущности.

## Configuring collection type factories

По умолчанию CycleORM использует `Cycle\ORM\Collection\ArrayCollectionFactory` для хранения данных связей.

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

В случае если вы хотите использовать другой тип колекций, то вам снача необходмо зарегистрировать фабрику:

```php
use Cycle\ORM;

$schema = new ORM\Schema(...);

$factory = (new ORM\Factory(
    dbal: $dbal,
    defaultCollectionFactory: new ORM\Collection\ArrayCollectionFactory    // Фабрика коллекций по умолчанию
))
    // Требуется установка пакета doctrine/collections
    ->withCollectionFactory(
        'doctrine',                                         // Алиас
         new ORM\Collection\DoctrineCollectionFactory,
         \Doctrine\Common\Collections\Collection::class    // <= Интерфейс или базовый класс коллекции
    )
    
    // Требуется установка пакета`illuminate/collections
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

> Метод `Factory::withCollectionFactory` является иммутабельными и при добавлении новых фабрик необходимо

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

$container->bindSingleton(
    ORM\ORMInterface::class, 
    $orm->with(factory: $factory)
);
```

## Указание коллекци для связей сущности.

### Определение через схему

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
                Relation::COLLECTION_TYPE => null,                  // <= Будет исопльзована коллекция по умолчанию
                Relation::SCHEMA => [ /*...*/ ],
            ],
            'comments' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Comment::class,
                Relation::COLLECTION_TYPE => 'doctrine',           // <= Будет использована коллекция с алиас `doctrine`
                Relation::SCHEMA => [ /*...*/ ],
            ],
            'tokens' => [
                Relation::TYPE => Relation::HAS_MANY,
                Relation::TARGET => Token::class,
                Relation::COLLECTION_TYPE => \Doctrine\Common\Collections\Collection::class, // <= Совпадение по базовому классу коллекции
                Relation::SCHEMA => [ /*...*/ ],
            ]
        ]
    ],
    Post::class => [
        //...
        Schema::RELATIONS   => [
            'comments' => [
                Relation::TYPE   => Relation::HAS_MANY,
                Relation::TARGET => Comment::class,
                Relation::COLLECTION_TYPE => CommentsCollection::class,    // <= Совпадение по классу, который наследует базовый класс
                Relation::SCHEMA => [ /*...*/ ],
            ]
        ]
    ]
];
```

### Определение через аннотации/атрибуты
```php
use Doctrine\Common\Collections\ArrayCollection;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasOne;
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\Annotated\Annotation\Relation\ManyToMany;

#[Entity]
class User
{
    #[Column(type: "primary")]
    protected $id;
    
    #[HasOne(target: Profile::class, load: "eager")]
    protected $profile;
    
    #[HasMany(target: Friend::class, load: "eager")]
    protected array $friends = [];
    
    #[HasMany(target: Profile::class, load: "eager", collection: 'doctrine')]
    protected ArrayCollection $posts;
   
    #[ManyToMany(target: Tag::class, load: through=TagMap::class, load: "lazy", collection: CommentsCollection::class)]
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
$users = $orm->getRepository(User::class)
    ->select()
    ->with('posts')->limit(1)->fetchOne();

print_r($u->posts);
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
