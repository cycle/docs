# Promise mapper

Cycle ORM provides the ability to carry data over the specific class instances by using `cycle/orm-promise-mapper`
package with `\Cycle\ORM\Reference\Promise` objects for relations with lazy loading.

## Installation

The preferred way to install this package is through [Composer](https://getcomposer.org/):

```bash
composer require cycle/orm-promise-mapper
```

## Define the Schema

:::: tabs

::: tab User

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;
use Cycle\ORM\PromiseMapper\PromiseMapper;
use Cycle\ORM\Reference\ReferenceInterface;

#[Entity(mapper: PromiseMapper::class)]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[HasMany(target: Post::class, collection: 'array', load: 'eager')]
    public array $posts;
    
    #[HasMany(target: Tag::class, load: 'lazy')]
    public ReferenceInterface|array $tags;
}
```

:::

::: tab Post

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\BelongsTo;
use Cycle\ORM\PromiseMapper\PromiseMapper;
use Cycle\ORM\Reference\ReferenceInterface;

#[Entity(mapper: PromiseMapper::class)]
class Post
{
    // ...

    #[BelongsTo(target: User::class, load: 'lazy')]
    public ReferenceInterface|User $user;

    #[BelongsTo(target: Tag::class, load: 'eager')]
    public Tag $tag;
}
```

:::

::::

## Fetching entity data

```php
$user = $orm->getRepository('user')->findByPK(1);

// $user->posts contains an array because of eager loading
foreach ($user->posts as $post) {
    // ...
}

// $user->tags contains Cycle\ORM\Reference\Promise object because of lazy loading
$tags = $user->tags->fetch();
foreach ($tags as $post) {
    // ...
}

$post = $orm->getRepository('post')->findByPK(1);

// $post->user contains Cycle\ORM\Reference\Promise object because of lazy loading
$userId = $post->user->fetch()->id;

// $post->tag contains Tag object because of eager loading
$tagName = $post->tag->name;
```
