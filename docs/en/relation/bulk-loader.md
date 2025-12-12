# Relation Bulk Loading

When you fetch multiple entities and need to load their relations, doing it one by one causes the N+1 query problem.
BulkLoader solves this by loading relations for all entities at once.
Only unloaded relations are resolved - already loaded relations won't be overwritten.

## Getting BulkLoader Instance

To load relations, use the `\Cycle\ORM\Relation\BulkLoaderInterface`.

If you're using a Cycle ORM bridge, get the BulkLoader instance from the DI container:

```php
use Cycle\ORM\Relation\BulkLoaderInterface;

final class Service
{
    public function __construct(
        private BulkLoaderInterface $bulkLoader
    ) {}
}
```

If you're not using a bridge, configure the binding in your application container:

```php
// Container configuration
$container->bindSingleton(
    \Cycle\ORM\Relation\BulkLoaderInterface::class,
    \Cycle\ORM\Relation\BulkLoader::class
);
```

Then inject it into your services as shown above.

## Basic Usage

```php
$users = $userRepository->findAll();

$bulkLoader
    ->collect(...$users)
    ->load('profile')
    ->run();

foreach ($users as $user) {
    echo $user->profile->imageUrl; // No extra queries
}
```

This executes just 2 queries instead of 1 + N: one to fetch users, one to fetch all their profiles.

## Multiple and Nested Relations

```php
// Multiple relations
$bulkLoader
    ->collect(...$users)
    ->load('profile')
    ->load('comments')
    ->load('posts')
    ->run();

// Nested relations
$bulkLoader
    ->collect(...$users)
    ->load('posts.comments')
    ->run();

// With options
$bulkLoader
    ->collect(...$users)
    ->load('comments', [
        'orderBy' => ['created_at' => 'DESC'],
        'where' => ['approved' => true]
    ])
    ->run();

// Sort by pivot columns (many-to-many)
$bulkLoader
    ->collect(...$users)
    ->load('tags', ['orderBy' => ['@.@.created_at' => 'DESC']])
    ->run();
```

## Collecting Entities

The `collect()` method is immutable and returns a new instance. You can chain calls to gather entities from different sources:

```php
$bulkLoader
    ->collect(...$activeUsers)
    ->collect(...$inactiveUsers)
    ->load('profile')
    ->run();
```

## Important Behavior

**All entities must be of the same type:**
```php
$bulkLoader
    ->collect($user, $post) // InvalidArgumentException
    ->run();
```

**Entities must exist in heap:**
```php
$user = new User(); // not persisted or fetched

$bulkLoader
    ->collect($user)
    ->load('profile')
    ->run(); // LogicException
```

**Relations use database state, not runtime changes:**
```php
$profile = $profileRepository->findByPK(1); // user_id = 5 in DB
$profile->user_id = 10; // changed at runtime

$bulkLoader->collect($profile)->load('user')->run();

// $profile->user still points to User(5), not User(10)
```

BulkLoader reads relation keys from heap node data (as fetched from database) to avoid inconsistencies.

**Already loaded relations are not overwritten:**
```php
$user = $userRepository->findByPK(1);
$user->profile; // lazy loading triggered

$bulkLoader->collect($user)->load('profile')->run();

// $user->profile is the same instance, not reloaded
```

## Examples

API responses:
```php
$posts = $postRepository->findRecent(50);

$bulkLoader
    ->collect(...$posts)
    ->load('author.profile')
    ->load('comments', ['orderBy' => ['created_at' => 'DESC']])
    ->run();

return $this->json($posts);
```

Reports:
```php
$orders = $orderRepository->findByMonth($month);

$bulkLoader
    ->collect(...$orders)
    ->load('items.product')
    ->load('customer.address')
    ->run();
```

Polymorphic relations:
```php
$images = $imageRepository->findAll();

$bulkLoader
    ->collect(...$images)
    ->load('parent') // User or Post
    ->run();
```
