# Collections
Cycle ORM use [Doctrine/Collection](https://github.com/doctrine/collections) in order to represent one to many relation types (such as hasMany, manyToMany).

## Accessing collection
ORM will automatically instantiate collection instance for your relations, however you still required to initiate empty
collections in your contructor to use newly created entities:

```php
/** @entity */ 
class User 
{
    // ...
    
    /** @hasMany(target = "Post") */
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
You can read more about available collection API [here](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/index.html).
