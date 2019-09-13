# Limitations
Current implementation of Cycle ORM includes multiple limitations.

## Compound Primary Keys
ORM relies on a unique primary key to create a proper entity map, using complex primary keys is currently not implemented.

See https://github.com/cycle/orm/issues/22

## Filter by relations from external databases
It is currently not possible to automatically filter the selection based on values of related entities located in an external source. 
You must manually filter the selected result after loading all the data. Though, it is possible to use `fetchData` of
`Cycle\ORM\Select` to avoid entity instantiation before the filtering.

Since you can provide any iterable source to the `Cycle\ORM\Iterator` you can create nested generator:

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

> Given approach will minimize the amount of allocated memory.

## MyISAM
It is not reliable to use Cycle with MySQL MyISAM engine as it does not support transaction which can guarantee the recovery from persisting errors. Use InnoDB engine instead.

## Cascade = false
Please note that turning cascade option off completelly disables relation `store` sequence. This makes uni-directional relations useless, only use this option if the relation is considered "read-only".

## Select->fetchOne() behaviour
Method `fetchOne` of Select will create query without specified `LIMIT` value in order to avoid data corruption on joined data. Make sure to manually set the limit or use proper selection constraint.

> `LIMIT 1` is set in Repository `findOne()`.
