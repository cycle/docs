# Direct Database Queries
You can always talk to the database directly without any ORM abstraction on top of it.

## Get Access to Entity Database
In order to receive access to the database specific to the entity you can either call `$dbal->database()` or request it via the Source object
available through the ORM:

```php
$source = $orm->getSource(User::class);

$db = $source->getDatabase();
$table = $source->getTable();
```

> Typecast ORM to `Cycle\ORM\Select\SourceProviderInterface` in order to ensure access to `getSource` method.

## Raw Queries
Once you gain access to the entity database you can write queries manually using the `query` method of the database:

```php
$result = $db->query('SELECT * FROM users WHERE id = ?', [1]);
print_r($result->fetchAll());
```

You can also use named parameters:

```php
$result = $db->query('SELECT * FROM users WHERE id = :id', [
  ':id' => 1
]);
print_r($result->fetchAll());
```

To execute a non-SELECT query:

```php
$affected = $db->execute('DELETE FROM users WHERE id = :id', [
  ':id' => 1
]);
print_r($affected);
```
