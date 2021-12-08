# Caching

Cycle ORM does not provide caching abilities on the core level, however, it provides multiple points at which caching
can be integrated.

## Caching the Selection

In order to cache the selected entities and their relations, you can use the method `fetchData` of `Cycle\ORM\Select`.

```php
$cacheStore = // ...

$userData = $userRepository->select()->load('profile')->fetchData();

$cacheStore->set('user-data', $userData);
```

The resulted array will contain all raw entity data (table columns) with typecasted values. You can then store this
information in a cache.

> Make sure that all custom column types are serializable.

In order to unpack cached information simply push given data into `Cycle\ORM\Iterator` object:

```php
$userData = $cacheStore->get('user-data');

$users = new Iterator($orm, User::class, $userData);
```

## Caching in Heap

Every loaded entity will be automatically placed into the Heap. The primary method to check if an object is already
located in the heap is `find`:

```php
$user = $orm->getHeap()->find(User::class, ['id' => 1]);

if ($user !== null) {
    // Do something 
}
```

Implementing your own Heap might provide you an ability to automatically store some objects in the cache.

```php
use Cycle\ORM\Heap\HeapInterface;

final class CachedHeap implements HeapInterface, \IteratorAggregate
{
    public function __construct(private CacheStore $cacheStore) {}
    
    // ...
}

$orm = $orm->with(heap: new CachedHeap(new CacheStore()))
```

> Cache invalidation can be achieved using a custom Mapper (persister) implementation by altering methods
> `queueUpdate` and `queueDelete`.
