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
    var_dump($table->getName());
}
```

> Only tables specific to the database prefix (if any) are returned.

Schema Reader/Builder (`AbstractTable`) is available using `getSchema` method:

```php
foreach ($database->getTables() as $table) {
    var_dump($table->getSchema());
}
```

## Reading table properties using AbstractTable

The AbstractTable provides low-level access to table information, such as column types (internal and abstract), indexes,
foreign keys, etc. You can use this information to perform a database export, or to build your own ORM or migration
mechanism (see [schema declaration](/docs/en/database/declaration.md)).

Table primary keys:

```php
var_dump($schema->getPrimaryKeys());
```

Table indexes:

```php
foreach ($schema->getIndexes() as $index) {
    var_dump($index->getName());
    var_dump($index->getColumns());
    var_dump($index->isUnique());
}
```

Table foreign keys (references):

```php
foreach ($schema->getForeigns() as $foreign) {
    var_dump($foreign->getColumn());       //Local column name
    var_dump($foreign->getForeignTable()); //Global table name!
    var_dump($foreign->getForeignKey());

    var_dump($foreign->getDeleteRule());   //NO ACTION, CASCADE
    var_dump($foreign->getUpdateRule());   //NO ACTION, CASCADE
}
```

> Attention, `getForeignTable` returns full table name ignoring db prefix.

Table columns:

```php
foreach ($schema->getColumns() as $column) {
    var_dump($column->getName());

    var_dump($column->getType());          //Internal database type
    var_dump($column->abstractType());     //Abstract type like string, bigInt, enum, text and etc.
    var_dump($column->phpType());          //PHP type: int, float, string, bool

    var_dump($column->getDefaultValue());  //Can be instance of SqlFragment

    var_dump($column->getSize());          //Only for strings and decimal values

    var_dump($column->getPrecision());     //Decimals only
    var_dump($column->getScale());         //Decimals only

    var_dump($column->isNullable());
    var_dump($column->getEnumValues());    //Only for enums

    var_dump($column->getConstraints());

    var_dump($column->sqlStatement());     //Column creation syntax
}
```

> Some types can be mapped incorrectly if the table was created outside migrations or ORM.

You can find a complete list of available abstract types [here](/docs/en/database/declaration.md).
