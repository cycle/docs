# Database Schema Introspection

Cycle/Database layer provides the ability to read and analyze the basic properties of a given database or a given table.
The DBAL layer includes a set of "abstract" types assigned to each column, based on DBMS specific mapping in order to
unify different engines.

## List of database tables

To check if the database has a specific table use `hasTable`:

```php
if ($database->hasTable('users')) {
    //...
}
```

> Read how to get Database instances [here](/docs/en/database/access.md).

Receive all database tables (array of `Cycle\Database\Table`):

```php
foreach ($database->getTables() as $table) {
    print_r($table->getName());
}
```

> Only tables specific to the database prefix (if any) are returned.

Schema Reader/Builder (`AbstractTable`) is available using `getSchema` method:

```php
foreach ($database->getTables() as $table) {
    print_r($table->getSchema());
}
```

## Reading table properties using AbstractTable

The AbstractTable provides low-level access to table information, such as column types (internal and abstract), indexes,
foreign keys, etc. You can use this information to perform a database export, or to build your own ORM or migration
mechanism (see [schema declaration](/docs/en/database/declaration.md)).

Table primary keys:

```php
print_r($schema->getPrimaryKeys());
```

Table indexes:

```php
foreach ($schema->getIndexes() as $index) {
    print_r($index->getName());
    print_r($index->getColumns());
    print_r($index->isUnique());
}
```

Table foreign keys (references):

```php
foreach ($schema->getForeigns() as $foreign) {
    print_r($foreign->getColumn());       //Local column name
    print_r($foreign->getForeignTable()); //Global table name!
    print_r($foreign->getForeignKey());

    print_r($foreign->getDeleteRule());   //NO ACTION, CASCADE
    print_r($foreign->getUpdateRule());   //NO ACTION, CASCADE
}
```

> Attention, `getForeignTable` returns full table name ignoring db prefix.

Table columns:

```php
foreach ($schema->getColumns() as $column) {
    print_r($column->getName());

    print_r($column->getType());          //Internal database type
    print_r($column->abstractType());     //Abstract type like string, bigInt, enum, text and etc.
    print_r($column->phpType());          //PHP type: int, float, string, bool

    print_r($column->getDefaultValue());  //Can be instance of SqlFragment

    print_r($column->getSize());          //Only for strings and decimal values

    print_r($column->getPrecision());     //Decimals only
    print_r($column->getScale());         //Decimals only

    print_r($column->isNullable());
    print_r($column->getEnumValues());    //Only for enums

    print_r($column->getConstraints());

    print_r($column->sqlStatement());     //Column creation syntax
}
```

> Some types can be mapped incorrectly if the table was created outside migrations or ORM.

You can find a complete list of available abstract types [here](/docs/en/database/declaration.md).
