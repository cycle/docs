# Usage in Long-Running Applications

Cycle ORM attempts to simplify the usage of the library in daemonized applications such as PHP workers running
under [RoadRunner](https://roadrunner.dev/), Swoole and etc. The ORM provides you multiple options to avoid memory
leaks (the same approach can be used for batch operations).

## Connection Configuration

Make sure to enable `reconnect` option in your database connection. Read more about database
configuration [here](/docs/en/database/connect.md).

## Cloning ORM

The first approach is based on the idea of creating separate ORM instances for each user request, each cloned ORM will
have its own `Heap`, which will be erased automatically by PHP GC:

```php
// you can also use scope specific factory and other options
$orm = $orm->with(heap: new \Cycle\ORM\Heap\Heap());
```

## Resetting the Heap

The alternative is to use a single ORM instance across all user requests, but reset the heap state after each iteration:

```php
// do something with orm
while ($action = getAction()) {
    do{($action);}

    $orm->getHeap()->clean();
}
```

ORM mappers and relations cache will remain intact, speeding up the application on next consecutive requests.

> Please note, cleaning the Heap does not commit any changes to the database, you must use Transaction for that.

## Batch operations

An identical approach can be used while working with batch data sets:

```php
$users = $orm->getRepository(User::class)->select();
for ($i = 0; $i < 100; $i++) {
    $users = $users->offset($i * 1000)->limit(1000)->fetchAll();

    $manager = new \Cycle\ORM\EntityManager($orm);
    foreach ($users as $user) {
        // Do something with user
        $user->ban();
        $manager->persist($user);
    }
    
    $manager->run();
}

$manager->clean(cleanHeap: true);
```

> You can combine `clone` and `reset` in order to create separate ORM instance for batch operations but keep all
> already loaded entities intact.

## Handling Exceptions

In some cases, you might experience the connection drop to your database. If the disconnect happens outside the
transaction, Cycle\Database will attempt to automatically reconnect. However, connection issues during the transaction
will throw an `Cycle\Database\Exception\DatabaseException` (more
specifically `Cycle\Database\Exception\Statement\ConnectionException`) exception.

Failures in the transaction would not affect ORM Heap (EntityManager). But the transaction will be clean. You can
reassemble the transaction and try again.

[//]: # (TODO проверить пример)

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($entity);
$state = $manager->run(throwException: false);

while ($error = $state->getLastError()) {
    if ($error instanceof \Cycle\Database\Exception\StatementException\ConnectionException) {
        usleep(100);
        $state->retry();
        continue;
    }
    
    throw $error;
}
```

> ORM Commands does not limit you to work in SQL scope, Transaction commands must implement the `execute`, `complete` 
> and `rollback` methods to support custom commit/compensate strategies, you can use a transaction to sync data across
> distributed services.
