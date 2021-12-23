# Entity Iterator

In order to initiate a set of entities and the tree of their relations Cycle ORM utilizes the `Cycle\ORM\Iterator`
class, which works in generator mode.

## Select Results

By default, the Iterator object is returned from the `getIterator` method of the `Cycle\ORM\Select` class.

```php
$select = $orm->getRepository('user')->select();

foreach ($select as $user) {
    print_r($user);
}
```

## Manual Iteration

Is it possible to provide input data to iterator manually, using any custom data source or raw SQL query? The provided 
collection must be a tree of entities data.

> Since Cycle ORM works with entity state using dirty state approach it is possible to load results partially (if default 
> entity values are null and does not trigger updates).

```php
$data = [
    ['id' => 1, 'name' => 'Antony'],
    ['id' => 2, 'name' => 'John']
];

$iterator =  \Cycle\ORM\Iterator::createWithOrm($orm, 'user', $data);
```

By default, iterator object requires prepared data collection (cast to proper types) and won't cast them automatically. 
If you pass a raw data collection (without typecasting) you have to set `typecast` argument to `true`.

```php
$iterator = \Cycle\ORM\Iterator::createWithOrm($orm, 'user', $rawData, typecast: true);
```

Also, it is possible to create an Iterator with the required services, instead of an ORM object:

```php
$iterator = \Cycle\ORM\Iterator::createWithServices($heap, $schema, $entityProvider, $role, $data);
```

## Pre-filtering

In some cases, you might want to filter selection results using external data sets (for example relations pointing to
the external database). Since filtering such results is not possible on the database level (using joins), you might want
to filter results internally, inside your PHP application.

In order to avoid additional memory consumption for objects, you can filter your results in generator mode prior to
model instantiation. To do that using the `Cycle\ORM\Select` method `fetchData`:

```php
use Cycle\ORM;

function filterByExternal(ORM\Select $select, $value): \Generator
{
    foreach($select->load('external')->fetchData() as $item) {
        if ($line['external']['value'] == $item) {
            yield $item;
        }
    }
}

// ...

foreach (new ORM\Iterator($orm, User::class, filterByExternal($select, $value)) as $user) {
    print_r($user);
}
```

> You can combine this approach with the `where` option of `load` method to create complex cross-database queries.
