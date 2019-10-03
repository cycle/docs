# Pagination
You can paginate any of the `Cycle\ORM\Select` objects using two basic methods: `limit` and `offset`. In addition to that,
the package `spiral/pagination` defines a set of basic interfaces which will allow you to easier integrate Cycle into your application.

## Example
To paginate a simple User select:

```php
$select = $orm->getRepository(User::class)->select()->orderBy('id', 'DESC');

// 10 results per page
$paginator = new Paginator(10);
$paginator->paginate($select);

print_r($select->fetchAll());
```

## Counting results
The Select instance implements the `Countable` interface and can be used to calculate the number of records prior to the selection:

```php
print_r($select->count());
```

## Paginating with relations
Please note that pagination happens on the database end. This forces you to explicitly set the result set as `distinct` if relations
like `hasMany`, `manyToMany` are joined to your query:

```php
$select = $orm->getRepository(User::class)->select()->orderBy('id', 'DESC');

$select->distinct()->with('posts');

// 10 results per page
$paginator = new Paginator(10);
$paginator->paginate($select);

print_r($select->fetchAll());
```
