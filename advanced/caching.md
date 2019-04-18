# Caching
Cycle ORM does not provide caching abilities on core level, however, it provides an convinient point at which caching can be intergated.

## Caching the Selection
In order to cache the selected enties and their relations you are able to utilize method `fetchData` of `Cycle\ORM\Select`.

```php
$userData = $userRepository->select()->load('profile')->fetchData();
```

The resulted array will contain all raw entity data (table columns) with typecasted values. You are able to store this information
in cache.

> Make sure that all custom column types are serializable.

In order to unpack cached information simply push given data into `Cycle\ORM\Iterator` object:

```php
$users = new Iterator($orm, User::class, $userData);
```

## Caching in Heap
Every loaded entity will be automatically placed into Heap. The primary method to check if object is already localed in heap is `find`:

```php
$u = $orm->getHeap()->find(User::class, 'id', 1);
```

Implementing your own Heap might provide you an ability to automatically store some objects in the cache.

> Cache invalidation can be achieved using custom Mapper (persister) implementation by altering methods `queueUpdate` and `queueDelete`.
