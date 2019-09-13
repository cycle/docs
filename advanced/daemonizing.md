# Usage in Long-Running Applications
Cycle ORM attempts to simplify the usage of the library in daemonized applications such as PHP workers running under RoadRunner, Swoole and etc.
ORM provides you multiple options to avoid memory leaks (the same approach can be used for batch operations).

## Connection Configuration
Make sure to enable `reconnect` option in your database connection. Read more about database configuration [here](/basic/connect.md).

## Cloning ORM
The first approach is based on the idea of creating separate ORM instance for each user request, each of cloned ORM will have it's own
`Heap` which will be erased automatically by PHP GC:

```php
// you can also use scope specific factory and other options
$orm = $orm->withHeap(new Heap());
```

## Resetting the Heap
Alternative is to use single ORM instance across all user requests but reset the heap state after each iteration:

```php
// do something with orm
while ($action = getAction()) {
    do($action);
    
    $orm->getHeap()->clean();
}
```

ORM mappers and relations cache will remain intact, speeding up the application on next consecutive request.

> Please note, cleaning the Heap does not commit any changes to the database, you must use Transaction for that.

## Batch operations
Identical approach can be used while working with batch data sets:

```php
$users = $orm->getRepository(User::class)->select();
for ($i = 0; $i < 100; $i++) {
    $users = $users->offset($i*1000)->limit(1000)->fetchAll();
  
    $t = new Transaction($orm);
    foreach ($users as $u) {
        $u->status = 'disabled';
        $t->persist($u);
    }
    $t->run();

    $orm->getHeap()->clean();
}
```

> You can combine `clone` and `reset` in order to create separate ORM instance for batch operations but keep all already loaded entities intact.

## Handling Exceptions
In some cases, you might experience the connection drop to your database. If the disconnect happens outside of the transaction Spiral\Database will attempt to automatically reconnect. However, connection issues during the transaction would throw exception `Spiral\Database\Exception\DatabaseException` (more specifically `Spiral\Database\Exception\Statement\ConnectionException`).

Failures in the transaction would not affect ORM Heap (EntityManager), this allows you manually reconnect to the database and retry:

```php
try {
   $t->run();
} catch (ConnectionException $e) {
   sleep(1);
   
   // try again?
   $t->run();
}
```

> ORM Commands does not limit you to work in SQL scope, Transaction commands must implement `execute`, `complete` and `rollback` methods
to support custom commit/compensate strategies, you can use a transaction to sync data across distributed services.
