# Custom Repositories

It is essential to be able to associate custom repository implementations to your entities. You are able to create any
repository from scratch, especially for non SQL sources. However, the most common use case would be extending the
default repository implementation to add custom selectors.

## Create Repository

To create a custom repository associated with a SQL data source simply extend the primary
class `Cycle\ORM\Select\Repository`:

```php
namespace Example\Repository;

class UserRepository extends \Cycle\ORM\Select\Repository
{
    // ...
}
```

Use the `Entity` attribute to create the association:

```php
namespace Example;

use Cycle\Annotated\Annotation as Cycle;

#[Cycle\Entity(repository: Example\Repository\UserRepository::class)]
class User
{
    // ...
}
```

> **Note**
> This applies to the `annotated` extension only. Other schema declaration approaches will differ in implementation.

Update/calculate your schema to get access to the newly assigned repository through the `getRepository` method of the
orm:

```php
$repository = $orm->getRepository(\Example\User::class);

print_r($repository::class);
```

> **Note**
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

> **Note**
> You can also chain your select methods `$this->findActive()->where('age', '>', $age);` as long as you return the `Select`
> object from your method.

Now you can access this method:

```php
print_r($orm->getRepository(\Example\User::class)->findActive()->fetchAll());
```

> **Warning**
> Don't mutate the `Repository::$select` object. Always use `Repository::select()` method to get a Select object clone.
> You can mutate the `$select` property only in a repository clone
> (see [Chained Repository](../advanced/chained-repository.md)).

## Preloading relations

Another use-case is to automatically preload some entity relations using a custom find method:

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
