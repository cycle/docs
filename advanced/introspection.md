# Database Schema Introspection
Spiral/Database layer provides the ability to read and analyze basic properties of a given database or a given table. DBAL layer include set of "abstract" types assigned to each column based on DBMS specific
mapping in order to unify different engines.

## List of database tables
To check if database has table use `hasTable`:

```php
if ($database->hasTable('users')) {
    //...
}
```

> Read how to get Database instances [here](/basic/connect.md).

Receive all database tables (array of `Spiral\Database\Table`):

```php
foreach ($database->getTables() as $table) {
    dump($table->getName());
}
```

> Only tables specific to database prefix (if any) are resulted.

Schema Reader/Builder (`AbstractTable`) is available using `getSchema` method:

```php
foreach ($database->getTables() as $table) {
    dump($table->getSchema());
}
```

## Reading table properties using AbstractTable
The AbstractTable provides low level access to table information such as column types (internal and abstract), indexes, foreign keys and etc. You can use this information to perform database export, build your own ORM or migration mechanism (see [schema declaration](/advanced/declaration.md)).

Table primary keys:

```php
dump($schema->getPrimaryKeys());
```

Table indexes:

```php
foreach ($schema->getIndexes() as $index) {
    dump($index->getName());
    dump($index->getColumns());
    dump($index->isUnique());
}
```

Table foreign keys (references):

```php
foreach ($schema->getForeigns() as $foreign) {
    dump($foreign->getColumn());       //Local column name
    dump($foreign->getForeignTable()); //Global table name!
    dump($foreign->getForeignKey());

    dump($foreign->getDeleteRule());   //NO ACTION, CASCADE
    dump($foreign->getUpdateRule());   //NO ACTION, CASCADE
}
```

> Attention, `getForeignTable` returns full table name ignoring db prefix.

Table columns:

```php
foreach ($schema->getColumns() as $column) {
    dump($column->getName());

    dump($column->getType());          //Internal database type
    dump($column->abstractType());     //Abstract type like string, bigInt, enum, text and etc.
    dump($column->phpType());          //PHP type: int, float, string, bool

    dump($column->getDefaultValue());  //Can be instance of SqlFragment
    
    dump($column->getSize());          //Only for strings and decimal values

    dump($column->getPrecision());     //Decimals only
    dump($column->getScale());         //Decimals only

    dump($column->isNullable());
    dump($column->getEnumValues());    //Only for enums

    dump($column->getConstraints());

    dump($column->sqlStatement());     //Column creation syntax
}
```

> Some types can be mapped incorrectly if the table was created outside migrations or ORM.

You can find a complete list of available abstract types [here](/advanced/declaration.md).
