# Entity Iterator
In order to initiate a set of entities and the tree of their relations Cycle ORM utilizes `Cycle\ORM\Iterator` class which works
in generator mode.

## Select Results
By default, the Iterator object is returned from the `getIterator` method of the `Cycle\ORM\Select` class.

```php
$select = $orm->getRepository('user')->select();

foreach ($select as $u) {
    print_r($u);
}
```

## Manual Iteration
Is it possible to provide input data to iterator manually, using any custom data source or raw SQL query? The data must be provided in a tree for.

> Since Cycle works with entity state using dirty state approach is it possible to load results partially (if default entity values
are null and does not trigger updates).

```php
$data = [
    ['id' => 1, 'name' => 'Antony'],
    ['id' => 2, 'name' => 'John']
];

$iterator = new ORM\Iterator($orm, 'user', $data);
```

## Pre-Filtering
In some cases, you might want to filter selection results using external data sets (for example relations pointing to the external database).
Since filtering such results is not possible on database level (using joins) you might want to filter results internally, inside your PHP
application.

In order to avoid additional memory consumption for objects, you can filter your results in generator mode prior to model instantiation.
To do that use `Cycle\ORM\Select` method `fetchData`:

```php
use Cycle\ORM;

function filterByExternal(ORM\Select $select, $value)
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

> You can combine this approach with `where` option of `load` method to create complex cross-database queries.
