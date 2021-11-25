# Collections
Cycle ORM use [Doctrine/Collection](https://github.com/doctrine/collections) in order to represent one to many relation types (such as hasMany, manyToMany).

> See [https://github.com/cycle/orm/issues/24](https://github.com/cycle/orm/issues/24)

## Accessing Collection
The ORM will automatically instantiate a collection instance for your relations, however, you are still required to initiate empty
collections in your constructor to use newly created entities:

```php
use Doctrine\Common\Collections\ArrayCollection;
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\HasMany;

#[Entity]
class User
{
    // ...

    #[HasMany(target: Post::class)]
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
You can read more about the available collection API [here](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/index.html).
