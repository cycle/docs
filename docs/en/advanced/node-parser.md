# Node Parser

Cycle ORM provides a convenient way to convert flat structures into data trees. The parser can work over one large query
or multiple queries using an identical approach. The parser works with numeric arrays.

> This section is intended for advanced scenarios, make sure you can't achieve required flexibility using default 
> instruments before jumping to this approach.

## Unpack simple query

We can start with a simple example which converts a query result into an associated array (example is using Database
instance):

```php
$query = $db->select('id', 'balance')->from('users');
```

The simple node parser will look like:

```php
use Cycle\Database\StatementInterface;
use Cycle\ORM\Parser\RootNode;

$root = new RootNode(
    ['id', 'balance'], // column names
    ['id']             // primary keys
);

foreach ($query->run()->fetchAll(StatementInterface::FETCH_NUM) as $row) {
    // start from 1st (0) column
    $root->parseRow(0, $row);
}

print_r($root->getResult());
```

The given code doesn't do much, but we can use it to perform more complex transformations. For example, we can join some
external table to our query:

```php
$query = $db
    ->select('u.id', 'u.balance', 'o.id', 'o.user_id', 'o.total')
    ->from('users as u')
    ->leftJoin('orders as o')->on('o.user_id', 'u.id');
```

The query will return results in a form: [user.id, user.balance, order.id, order.user_id, order.total]. Let's unpack it
into a structure like:

```php
[
    [
        'id' => 1,
        'balance' => 10,
        'orders' => [
            [
                'id' => 1,
                'user_id' => 1,
                'total' => 100,
            ],
            // ...
        ],
    ],
]
```

Since both tables are merged in one query we have to create and join sub-node (array):

```php
use Cycle\ORM\Parser;
use Cycle\Database\StatementInterface;

$root = new Parser\RootNode(
    ['id', 'balance'], ['id']
);

$root->joinNode('orders', new Parser\ArrayNode(
    ['id', 'user_id', 'total'], // column names
    ['id'],                     // primary keys
    ['user_id'],                // inner keys
    ['id']                      // outer keys (user.id)
));

foreach ($query->run()->fetchAll(StatementInterface::FETCH_NUM) as $row) {
    // start from 1st (0) column
    $root->parseRow(0, $row);
}
```

> Check SingularNode for one to one associations.

## External Queries

In some cases (for example for one-to-many associations) it might be useful to execute a relation query using external
SQL `SELECT` and `WHERE IN` statement. This can be achieved by linking nodes together to aggregate query context:

```php
use Cycle\ORM\Parser;
use Cycle\Database\StatementInterface;

$query = $db->select('u.id', 'u.balance')->from('users as u');

$root = new Parser\RootNode(
    ['id', 'balance'], ['id']
);

$orders = new Parser\ArrayNode(
    ['id', 'user_id', 'total'], // column names
    ['id'],                     // primary keys
    ['user_id'],                // inner keys
    ['id']                      // outer keys (user.id)
);

// notice the change
$root->linkNode('orders', $orders);

foreach ($query->run()->fetchAll(StatementInterface::FETCH_NUM) as $row) {
    // start from 1st (0) column
    $root->parseRow(0, $row);
}
```

Now, the `orders` array in our structure would not be populated, but we can request a list of collected ids from the
root loader:

```php
// only populated after parsing all the rows by the root node
print_r($orders->getReferences());
```

We can use this references (user.id) to create orders query:

```php
use Cycle\Database\Injection\Parameter;
use Cycle\Database\StatementInterface;

$query = $db
    ->select('o.id', 'o.user_id', 'o.total')
    ->from('orders as o')
    ->where('o.user_id', 'in', new Parameter($orders->getReferences()));

foreach ($query->run()->fetchAll(StatementInterface::FETCH_NUM) as $row) {
    // start from 1st (0) column
    $orders->parseRow(0, $row);
}
```

Order node will mount it's parsed entities to root node automatically, giving us identical result as in case
with `LEFT JOIN`:

```php
print_r($root->getResult();
```
