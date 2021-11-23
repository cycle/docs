# Database - Transactions
Both Database and Driver instances provide access to transaction management for your queries.
You can manage transactions manually or use pre-created Closure based flow.

## Examples
To start and commit transaction manually use `begin` and `commit` methods of Database:

```php
$db->begin();

// your queries

$db->commit();
```

Rollback:

```php
$db->begin();

// your queries

$db->rollback();
```

## Alternative Approach (recommended)
You can let Database manage state of your transaction automatically using `transaction` method:

```php
$db->transaction(function (Database $db) {
    //Queries    
});
```

> Transaction will be rolled back in case of any exception.

## Connection Errors
DBAL will attempt to reconnect to the database in case of connection drop only when no transactions open.
