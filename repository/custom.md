# Custom Repositories
It is essential to be able to associate custom repository implementations to your entities. You are able to create 
any repository from scratch, especially for non SQL sources. However, the most common use case would be extending default
repository implementation to add custom selectors.

## Create Repository
To create custom repository associated with SQL data source simply extend primary class `Cycle\ORM\Select\Repository`:

```php
namespace Example\Repository;

use Cycle\ORM\Select;

class UserRepository extends Select\Repository 
{

}
```

Use `entity` annotation attribute to create the association:

```php
namespace Example;

/**
 * @entity(repository="Example\Repository\UserRepository")
 */
class User 
{
  // ...
}
```

You can also specify repository name using relative namespace path:

```php
namespace Example;

/**
 * @entity(repository="Repository\UserRepository")
 */
class User 
{
  // ...
}
```

Update/calculate your schema to get access to newly assigned repository though `getRepository` method of orm:

```php
print_r(get_class($orm->getRepository(\Example\User::class)));
```

> You can assing one repository implementation to multiple entities.

## Custom Selects
Main reason of using custom repositories is the ability to write your own `find` methods. You can do that using 
base `select` method which return you instance of `Cycle\ORM\Select`:

```php
namespace Example\Repository;

use Cycle\ORM\Select; 

class UserRepository extends Select\Repository 
{
    public function findActive(): Select 
    {
        return $this->select()->where('status', 'active');
    }
}
```

> You can also chain your select methods `$this->findActive()->where('age', '>', $age);` as far as you return `Select`
object from your method.

Now you can access to this method:

```php
print_r($orm->getRepository(\Example\User::class)->findActive()->fetchAll());
```

## Preloading relations
Another use-case is to automatically pre-load some of entity relations using custom find method:

```php
class UserRepository extends Select\Repository 
{
    // ...

    public function findActiveUsersLoadAddress(): Select 
    {
        return $this->findActive()->load('address');
    }
}
```

## Integration with Spiral Framework
If you use Spiral Framework all custom repositories would be available directly in the container, without need to request $orm instance.

```php
public function listAction(UserRepository $users) 
{
    print_r($users->findActive()->fetchAll());
}
```

> Note, given approach will only work for unique entity-repository associations.
