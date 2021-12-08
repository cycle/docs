# Security

Cycle query builder will automatically pass all of the used parameters as part of the prepared statement. However, you
have to remember that column and table names have to be escaped on the application level.

## Identifiers

Avoid using user provided identifiers without a proper whitelist:

```php
$users = $userRepository->select();

$users->where($userParam, '=', $value); // possible SQL injection
```

Avoid using user values in `orderBy` as well:

```php
$users->orderBy($userParam, $userDirection); // possible SQL injection
```

> Same goes for aggregation methods `sum`, `avg`, `min`, `max`.

## Expression and Fragments

**No** user input must be used in `Fragment` and `Expression` wrappers:

```php
$users->where($name, '=', new \Cycle\Database\Injection\Expression("CONCAT($userValue)")); // possible SQL injection
```

## Array Parameters

The ORM will not allow you to use array parameters outside of `Parameter` scope:

```php
$users->where($id, 'IN', [1, 2, 3]); // compile exception
$users->where($id, 'IN', new \Cycle\Database\Injection\Parameter([1, 2, 3])); // valid approach
```
