# Complex Queries
You are able to use query builder to compose more complex queries and expression like conditions.

## Expressions
It is possible to inject custom SQL logic into the query using `Spiral\Database\Injection\Expression` object:

```php
$select->where('time_created', '>', new Expression("NOW()"));
```

You are able to use expressions in place of operators or column names. Please note, that since column name might not necessary be idential to the actual property name you must resolve it's identificator first.

## Name Resolver
To resolve name of column you must gain access to `QueryBuilder` instance available thought `getBuilder` method of `Select` object:

```php
$qb = $select->getBuilder();

print_r($qb->resolve('id')); // table.column_name
```

You can use this identificator inside your expressions:

```php
$qb = $select->getBuilder();

// to compare 2 columns
$select->where(
  'credits', 
  '>', 
  new Expression($qb->resolve('balance'))
);
```

Such query will produce similar SQL:

```sql
SELECT 
    ...
FROM "users" AS "user" WHERE 
  "user"."credits" > "user"."balance" 
```

You can also resolve names of related entities by using entity path:

```php
$select->distinct()
    ->where('balance', '>', new Expression($qb->resolve('orders.total')))
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

> You can also use methods `groupBy` on your `Select` object to create more complex conditions.

## Low level queries
If you want to run complex selection or select only particular columns you can modify underlying query direcly:

```php
$query = $select->buildQuery();

$query
    ->columns('id', new Expression('SUM(balance)'))
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

> Use `resolve` method to obtain fully qualified column names.

## Injecting Queries
It is possible to inject query into another query. In this case you must obtain an instance of entity query first, it can be done
by calling method `buildQuery()` of `Select` object.

For example:

```php
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

> Both entities must locate within one physical database.
