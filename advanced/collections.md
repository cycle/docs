# Collections
Cycle ORM use [Doctrine/Collection](https://github.com/doctrine/collections) in order to represent one to many relation types (such as hasMany, manyToMany).

> See https://github.com/cycle/orm/issues/24

## Accessing Collection
ORM will automatically instantiate collection instance for your relations, however, you still required to initiate empty
collections in your constructor to use newly created entities:

```php
/** @Entity */ 
class User 
{
    // ...
    
    /** @HasMany(target = "Post") */
    public $posts;
    
    public function __construct()
    {
        $this->address = new ArrayCollection();
    }
}
```

The collection properly will be set automatically on the selection:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('posts')->limit(1)->fetchOne();
    
print_r($u->posts);
```

## Collection API
You can read more about the available collection API [here](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/index.html).
