# DateTime

All properties with type "date", "datetime", "timestamp" will be represented using a DateTimeImmutable object.

By default, the ORM uses single DB specific timezone which can be configured on driver level, all dates selected from
and stored into the database will be converted into UTC timezone.

DBAL will automatically convert any DateTimeInterface parameter into the appropriate timezone to ensure proper data
selection.

> You can use [Behaviors](/docs/en/entity-behaviors/timestamps.md) for `created_at`, `updated_at` and `deleted_at` columns.
