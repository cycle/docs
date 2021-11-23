# Complex Queries
You can use the the query builder to compose more complex queries and expressions.

## Expressions
It is possible to inject custom SQL logic into the query using `Cycle\Database\Injection\Expression` object:

```php
$select->where('time_created', '>', new \Cycle\Database\Injection\Expression("NOW()"));
```

You can use expressions in place of operators or column names. Please note that, since a column name might not necessarily be identical to the actual property name, you must resolve its identity first.

## Name Resolver
To resolve the name of column you must gain access to `QueryBuilder` instance available through the `getBuilder` method of the `Select` object:

```php
$qb = $select->getBuilder();

print_r($qb->resolve('id')); // table.column_name
```

You can use this identification inside your expressions:

```php
$qb = $select->getBuilder();

// to compare 2 columns
$select->where(
  'credits',
  '>',
  new \Cycle\Database\Injection\Expression($qb->resolve('balance'))
);
```

Such a query will produce similar SQL:

```sql
SELECT
    ...
FROM "users" AS "user" WHERE
  "user"."credits" > "user"."balance"
```

You can also resolve names of related entities by using the entity path:

```php
$select->distinct()
    ->where('balance', '>', new \Cycle\Database\Injection\Expression($qb->resolve('orders.total')))
    ->andWhere('orders.status', 'pending');
```

Example SQL:

```sql
SELECT DISTINCT
   ...
FROM "users" AS "user"
INNER JOIN "orders" AS "user_orders"
    ON "user_orders"."user_id" = "user"."id"
WHERE "user"."balance" > "user_orders"."total" AND "user_orders"."status" = 'pending'
```

> You can also use the method `groupBy` on your `Select` object to create more complex conditions.

## Low level queries
If you want to run complex selection or select only particular columns you can modify the underlying query directly:

```php
$query = $select->buildQuery();

$query
    ->columns('id', new \Cycle\Database\Injection\Expression('SUM(balance)'))
    ->groupBy('id');

print_r($query->fetchAll());
```

The produced query will look like:

```sql
SELECT
  "id", SUM("balance")
FROM "users" AS "user"
  GROUP BY "id"
```

> Use the `resolve` method to obtain fully qualified column names.

## Injecting Queries
It is possible to inject a query into another query. In this case, you must obtain an instance of the entity query first. It can be done
by calling the method `buildQuery()` of `Select` object.

For example:

```php
use Cycle\Database\Injection\Expression;

$users = $orm->getRepository(User::class)->select();
$orders = $orm->getRepository(Order::class)->select();

// only orders of specific user (fallback to native column name)
$sumOrders = $orders->where('user_id', new Expression($users->getBuilder()->resolve('id')))->buildQuery();

$sumOrders->columns(new Expression('SUM('. $orders->getBuilder()->resolve('total') .')'));

$users->where(
    $sumOrders,
    '>=',
    new Expression($users->getBuilder()->resolve('balance'))
);
```

The produced SQL will look like:

```sql
SELECT
    ...
FROM "users" AS "user"
  WHERE (
    SELECT
    SUM("order"."total")
    FROM "orders" AS "order"
    WHERE "order"."user_id" = "user"."id"
  ) >= "user"."balance"
```

> Both entities must be located within one physical database.
