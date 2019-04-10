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

You can now associte this repository to your entinty by specifying `Schema::REPOSITORY` in your entity schema. Easier path would be 
using `entity` annotation for your class:

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

Update your schema to get access to newly assigned repository though `getRepository` method of orm:

```php
print_r(get_class($orm->getRepository(\Example\User::class)));
```

## Custom Selects
Main reason of using custom repositories is the ability to write your own `find` methods. You can do that using 
internal `find` or `select` method.

```php
namespace Example\Repository;

use Cycle\ORM\Select; 

class UserRepository extends Repository 
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
print_r($orm->getRepository(\Example\User::class)->findActive()->findAll());
```
