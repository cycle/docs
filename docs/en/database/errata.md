# Database - Errata

The documentation section lists non-obvious behaviors of the DBAL component and ways to avoid them.

## Timestamps

Consider using 'datetime' type for your columns instead of 'timestamp' as it gives you more unified support and ability
to define proper default values.

> Timestamp fields behavior is different in MySQL 5.5 and MySQL 5.6+!

## Driver Specific notes

Not all drivers implemented equally:

### MySQL Driver

The DBAL layer fully supports MySQL driver with all functionality being available.

The default table engine set to `InnoDB`.

### Postgres Driver

Postgres driver includes custom implementation of `InsertQuery` to address the return value of auto-incremental PK, it
will automatically add `RETURNING {primary key}` to the generated SQL query.

### SQLServer Driver

SQLServer includes a fallback mechanism to limit your selection without orderBy specified. In some cases, it might add
`_row_number_` column at the end of the selected columns.

> SQLServer is optimized to work with 12+ versions of [Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server/).

### SQLite Driver

SQLite DBMS does not support a set of table altering operations, to address some of the schema migrations will be
performed using temporary tables and data copy.
