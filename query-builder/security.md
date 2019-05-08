# Security
Cycle query builder will automatically pass all of the use parameters as part of the prepared statement. However, you have to remember that column and table names are escaped on application level. 

## Identifiers
Avoid using user provided identifiers without proper whitelist:

```php
$users = $userRepository()->select();

$users->where($userParam, '=', $value); // possible SQL injection
```

Avoid using user values in `orderBy` as well:

```php
$users->orderBy($userParam, $userDirection); // possible SQL injection
```

## Expression and Fragments
None of user input must be used in `Fragment` and `Expression` wrappers:

```php
$users->where($name, '=', new Expression("CONCAT($userValue)")); // possible SQL injection
```
