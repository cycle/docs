# Proxy Mapper

Cycle ORM provides the ability to carry data over the specific class instances by using `\Cycle\ORM\Mapper\Mapper`.
Every entity created by the ORM via `\Cycle\ORM\ORM::make()` method will be a proxy entity.

> The proxy entity performs "lazy loading" for properties with relations. It means that relation data will
> only be loaded when you actually access them.

Proxy Entities are based primarily on two Design Patterns:

- Proxy Pattern,
- Lazy Loading Pattern.

A proxy entity is an object that is put in place or used instead of the "real" object. A proxy entity can add behavior
to the object being proxied without that object being aware of it. In the ORM, proxy entity are used to realize several
features but mainly for transparent lazy-loading.

When you fetch an entity from a database, the ORM loads this entity as a proxied object, fully initialized, except the
entities that are associated with the proxy object through relations. Then, when we further access a method or property
of this proxied object, the ORM will make a request to the database to load that property if it’s not already loaded.

## Define the Entity

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\BelongsTo;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[HasMany(target: Post::class, load: 'eager')]
    public array $posts;
    
    #[HasMany(target: Tag::class, load: 'lazy')]
    public array $tags;
}

#[Entity]
class Post
{
    // ...

    #[BelongsTo(target: User::class, load: 'lazy')]
    public User $user;

    #[BelongsTo(target: Tag::class, load: 'eager')]
    public Tag $tag;
}
```

## Fetching entity data

```php
$user = $orm->getRepository('user')->findByPK(1);

foreach ($user->posts as $post) {
    // ...
}

foreach ($user->tags as $post) {
    // ...
}

$post = $orm->getRepository('post')->findByPK(1);

$userId = $post->user->id;

$tagName = $post->tag->name;
```

## Limitations:

Proxy usage applies a set of limitations on your entities:

- You cannot declare your entity as `final`.
- Don't use `get_class($user) === User::clas`, use `$entity instanceof User` instead.
- Create entity classes without considering it might be a proxy object. There can be used typed and private properties
  in entities. Properties with relations won't contain `Cycle\ORM\Reference\ReferenceInterface`, they will contain real
  types.
- Using lazy loaded relations as PHP links may not be possible because the magic method `__get()` can't return links.
  ```php
  $post->tags[] = new Tag(); // Indirect modification of overloaded property Post::$tags has no effect
  $comments = &$post->comments; // Indirect modification of overloaded property Post::$comments has no effect
  ```
  In this case you should preload a lazy relation:
  ```php
  // Load tags
  $post->tags;
  $post->tags[] = new Tag();
  // Load comments
  $post->comments;
  $comments = &$post->comments;
  ```

> Note: All proxy entities implement `\Cycle\ORM\EntityProxyInterface` interface.

