# DateTime
All properties with type "date", "datetime", "timestamp" will be represented using a DateTimeImmutable object.

By default the ORM uses single DB specific timezone which can be configured on driver level, all dates selected from and stored into the database will be converted into UTC timezone.

DBAL will automatically convert any DateTimeInterface parameter into the appropriate timezone to ensure proper data selection.

## Updates
You have to remember that ORM calculates entity difference based on column references, this means that you must only use immutable version of DateTime.

```php
$user->created_at->setDate(...); // error, won't trigger an update
```

Proper way:

```php
$user->created_at = new \DateTimeImmutable(...);
```
