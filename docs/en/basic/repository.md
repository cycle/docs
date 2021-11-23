# Custom Repositories
It is essential to be able to associate custom repository implementations to your entities. You are able to create any repository from scratch, especially for non SQL sources. However, the most common use case would be extending the default
repository implementation to add custom selectors.

## Create Repository
To create a custom repository associated with a SQL data source simply extend the primary class `Cycle\ORM\Select\Repository`:

```php
namespace Example\Repository;

class UserRepository extends \Cycle\ORM\Select\Repository
{

}
```

Use the `entity` annotation attribute to create the association:

```php
namespace Example;

use Cycle\Annotated\Annotation as Cycle;

/**
 * @Cycle\Entity(repository="Example\Repository\UserRepository")
 */
class User
{
  // ...
}
```

> This applies to the `annotated` extension only. Other schema declaration approaches will differ in implementation.

You can also specify the repository name using a relative namespace path:

```php
namespace Example;

use Cycle\Annotated\Annotation as Cycle;

/**
 * @Cycle\Entity(repository="Repository\UserRepository")
 */
class User
{
  // ...
}
```

Update/calculate your schema to get access to the newly assigned repository through the `getRepository` method of the orm:

```php
print_r(get_class($orm->getRepository(\Example\User::class)));
```

> You can assign a single repository implementation to multiple entities.

## Custom Selects
The main reason for using custom repositories is the ability to write your own `find` methods. You can do that using
base `select` method which returns you the instance of `Cycle\ORM\Select`:

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

> You can also chain your select methods `$this->findActive()->where('age', '>', $age);` as long as you return the `Select`
object from your method.

Now you can access this method:

```php
print_r($orm->getRepository(\Example\User::class)->findActive()->fetchAll());
```

## Preloading relations
Another use-case is to automatically pre-load some of the entity relations using a custom find method:

```php
use Cycle\ORM\Select;

class UserRepository extends Select\Repository
{
    // ...

    public function findActiveUsersLoadAddress(): Select
    {
        return $this->findActive()->load('address');
    }
}
```
