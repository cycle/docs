# DateTime
All properties with type "date", "datetime", "timestamp" will be represented using DateTimeImmutable object.

By default ORM use single DB specific timezone which can be configured on driver level, all dates selected from database and to be stored in
database will be converted into UTC timezone.

DBAL will automatically convert any DateTimeInterface parameter into appropriate timezone to ensure proper data selection.

## Updates
You have to remember that ORM calculate entity difference based on column references, this means that you must only use immutable version of DateTime.

```php
$user->created_at->setDate(...); // error, won't trigger an update
```

Proper way:

```php
$user->created_at = new DateTimeImmutable(...);
```
