# Database Schema Declaration
Spiral/Database ships with an included mechanism to declare table structures, FKS and indexes using declarative approach and schema comparison.

> Practically, table changes can be executed using an external migration system.

## Principle of Work
Before any operation/declaration can be applied to the table schema, DBAL will load the currently existing structure from the database and [normalize it into internal format](/advanced/introspection.md).

As a result, you are allowed to apply the modification to the table schema using a declarative way instead of an imperative one. Once schema **save** is requested, DBAL will generate a set of creation and altering operations based on the difference between the declared and existing schemas.

> See below how to use `Spiral\Database\Schema\Reflector` to sync multiple related tables.

## To Start
To get an instance of `AbstractTable` use a similar way as described in [Schema Introspection (make sure your read them first)](/advanced/introspection.md).

> No need to check for table existence.

```php
protected function indexAction(\Spiral\Database\Database $database)
{
    $schema = $database->table('new_table')->getSchema();

    // Schema is supposed to be empty
    dump($schema);
    dump($schema->exists());
}
```

## Columns and Abstract Types
You can add columns to a specific schema by simply setting their type. Use the following example to start:

```php
$schema = $database->table('new_table')->getSchema();

$schema->column('id')->primary();
$schema->column('name')->string(64); //String length 64 characters
$schema->column('email')->string();  //Default string length is 255 symbols
$schema->column('balance')->decimal(10, 2);
$schema->column('description')->text();
```

> All of the listed methods are added into the table and column doc comments so your IDE/editor will highlight them.

Use the shorter version if you find it easier:

```php
$schema = $database->table('new_table')->getSchema();

$schema->primary('id');
$schema->string('name', 64); //String length 64 characters
$schema->string('email');    //Default string length is 255 symbols
$schema->decimal('balance', 10, 2);
$schema->text('description');
```

To create the table schema in the database we have to call the method `save` of our AbstractTable:

```php
$schema->save();
```

Depending on which database driver you are using, DBAL will generate different SQL statements to create a table:

```sql
CREATE TABLE `primary_new_table` (
    `id` int (11) NOT NULL AUTO_INCREMENT,
    `name` varchar (64) NULL,
    `email` varchar (255) NULL,
    `balance` decimal (10, 2) NULL,
    `description` text NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
```

> Note that database prefix has been addressed automatically.

In Postgres the create syntax will look like:

```sql
CREATE TABLE "secondary_new_table" (
    "id" serial NOT NULL,
    "name" character varying (64) NULL,
    "email" character varying (255) NULL,
    "balance" numeric (10, 2) NULL,
    "description" text NULL,
    PRIMARY KEY ("id")
)
```

> Note, by default every column is created as nullable. Use the `nullable` method to overwrite it (see below).

Once the schema is created you can add new columns into it by only declaring them in your code:

```php
$schema = $database->table('new_table')->getSchema();

$schema->primary('id');
$schema->string('name', 64); //String length 64 characters
$schema->string('email');    //Default string length is 255 symbols
$schema->decimal('balance', 10, 2);

$schema->longText('description'); //New type
$schema->integer('count_visits'); //New column

$schema->save();
```

DBAL will detect 2 new declarations being added and generate appropriate code:

```sql
ALTER TABLE `primary_new_table` CHANGE `description` `description` longtext NULL;
ALTER TABLE `primary_new_table` ADD COLUMN `count_visits` int (11) NULL;
```

> Attention, not every type can be easily changed in some databases. Make sure you are not violating DBMS specific cross-type conversion (string => integer for example).


### Abstract Types
As you can notice, DBAL uses a set of "abstract" (common for all DBMS) types to declare table columns. Internally such types are mapped to appropriate internal DBMS column type.

Type        | Parameters                | Description
---         | ---                       | ---
**primary** | ---                       | Special column type, usually mapped as integer + auto-incrementing flag and added as table primary index column. You can define only one primary column in your table (you can still create a compound primary key, see below).
bigPrimary  | ---                       | Same as primary but uses `bigInteger` to store its values.
boolean     | ---                       | Boolean type, some databases store it as an integer (1/0).
integer     | ---                       | Database specific integer (usually 32 bits).
tinyInteger | ---                       | Small/tiny integer, check your DBMS to check its size.
bigInteger  | ---                       | Big/long integer (usually 64 bits), check your DBMS to check its size.
**string**  | [length:255]              | String with specified length, a perfect type for emails and usernames as it can be indexed.
text        | ---                       | Database specific type to store text data. Check DBMS to find size limitations.
tinyText    | ---                       | Tiny text, same as "text" for most of the databases. Differs only in MySQL.
longText    | ---                       | Long text, same as "text" for most of the databases. Differs only in MySQL.
double      | ---                       | [Double precision number.] (https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
float       | ---                       | Single precision number, usually mapped into "real" type in the database.
decimal     | precision,&nbsp;[scale:0] | Number with specified precision and scale.
datetime    | ---                       | To store specific date and time, DBAL will automatically force UTC timezone for such columns.
date        | ---                       | To store date only, DBAL will automatically force UTC timezone for such columns.
time        | ---                       | To store time only.
*timestamp* | ---                       | Timestamp without a timezone, DBAL will automatically convert incoming values into UTC timezone. Do not use such column in your objects to store time (use DateTime instead) as timestamps will behave very specific to select DBMS.
binary      | ---                       | To store binary data. Check specific DBMS to find size limitations.
tinyBinary  | ---                       | Tiny binary, same as "binary" for most of the databases. Differs only in MySQL.
longBinary  | ---                       | Long binary, same as "binary" for most of the databases. Differs only in MySQL.
json        | ---                       | To store JSON structures, usually mapped to "text", only Postgres supports it natively.

> Attention, in some cases the type returned by `ColumnSchema->abstractType()` might not be the same as declared one, such problem may occur in cases when DBMS uses the same internal type for multiple abstract types (for example most of the databases does not differentiate long/short/medium text and binary types).

> However, this  does not break anything in schema synchronization as DBAL creates operations based on the difference in internal database types, not based on the declared abstract one.

### Enum Type
The Enum type only exists natively in the MySQL database, in other DBMS it will be emulated using string type with associated constrain. To define enum type you have to list it's values:

```php
$schema->column('status')->enum(['active', 'disabled']);

//Alternative definition
$schema->enum('statusB', ['active', 'disabled']);
```

> As in other cases declared schema will be synced will database one, so you can add and remove enum values at any moment.

### Default values
It's recommended to set a default value for enum and some other columns. This can be performed using `defaultValue()`:

```php
$schema->column('status')->enum(['active', 'disabled'])->defaultValue('disabled');
$schema->enum('status_b', ['active', 'disabled'])->defaultValue('active');
```

And again, you can change the default value at any moment. If you wish to drop the default value simple set the method argument as `null`.

```php
$schema->enum('statusB', ['active', 'disabled'])->defaultValue(null);
```

### Nullable columns
To set column as NOT NULL use `nullable` method with `false` as parameter:

```php
$schema->string('name', 64)->nullable(false);
```

You can change NULL/NOT NULL flag at any moment you want. Additionally, you can try to combine NOT NULL column with the non-empty default value, this will allow you to add new columns to the non-empty table.

```php
$schema->integer('new_column')->nullable(false)->defaultValue(0);
```

> ORM will automatically resolve default value for NOT NULL casted columns.

## Primary Index
Table primary index can be set only while creation. DBAL will set PK automatically based on the column with type "primary" or "bigPrimary" declared in your schema.

To declare compound or custom primary keys, use table method `setPrimaryKeys()`.

```php
$schema->primary('id');
$schema->string('something', 16);
$schema->setPrimaryKeys(['id', 'something']);
```

> You are not able to change primary keys after the table being created.

## Indexes
Use methods `index` to declare indexes, array of column names is required:

```php
$schema = $database->table('other_table')->schema();

$schema->primary('id');
$schema->string('name', 64)->nullable(false);

$schema->string('email');

$schema->index(['email']); //Simple index
$schema->column('email')->index(); //You can also use alternative declaration for simple indexes

$schema->index(['name', 'email']); //Compound index

$schema->save();
```

If you wish to add unique index you can change your code a little bit:

```php
$schema->column('email')->unique(); //Simple unique index
$schema->index(['name', 'email'])->unique(true);  //Compound unique index
```

You can make index non unique at any moment:

```php
$schema->index(['name', 'email'])->unique(false);
```

> Attention, you can not add indexes to text or binary columns. You have to remember about limitations current DBMS applies to its indexes. For example, you can not create a unique index for the non-empty table with invalid (from the standpoint of the index) data. Some databases also have a maximum index size and etc.

## Foreign Keys
You can link tables together by using FKs:

```php
$first = $database->table('first')->getSchema();

$first->primary('id');
$first->string('name', 64);
$first->string('email');

$first->save();

$second = $database->table('second')->getSchema();

$second->bigPrimary('id');
$second->string('title');

$second->save();
```

In order to create FK we have to define local column and call `foreign` method on it:

```php
$second->integer('first_id');
$second->foreign('first_id')->references('first', 'id');
```

If we using MySQL connection DBAL will generate following SQL:

```sql
ALTER TABLE `primary_second` ADD COLUMN `first_id` int (11) NULL;
ALTER TABLE `primary_second` ADD CONSTRAINT `primary_second_foreign_first_id_55f205f594a3a` FOREIGN KEY (`first_id`) REFERENCES `primary_first` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION;
```

You can define custom DELETE and UPDATE rules for your FK:

```php
$foreignKey = $second->foreign('first_id')->references('first', 'id');

$foreignKey->onDelete(\Cycle\ORM\Reference\ReferenceInterface::CASCADE);
$foreignKey->onUpdate(\Cycle\ORM\Reference\ReferenceInterface::CASCADE);
```

Now, when the record in "first" table will be removed related data from the "second" table will be wiped also. You can read more about different actions [here](https://en.wikipedia.org/wiki/Foreign_key#Referential_actions).

> Please note that not every DBMS support actions outside of NO ACTION and CASCADE. In addition, some databases (hi, Microsoft) may forbid multiple foreign keys with CASCADE action in one table to avoid a reference loop.

## Rename Schemas
You can rename column in existed table by simply giving it new name or via shortcut method:

```php
$schema->string('email')->setName('new_email');
```

```php
$schema->renameColumn('email', 'new_email');
```

> Call `save` method of `AbstactTable` to save your changes.

Use similar approach to rename indexes and table name.

```php
$schema->setName('new_table_2');
$schema->save();
```

## Drop columns and indexes
Use methods `dropColumn`, `dropIndex` and `dropForeign` to remove elements:

```php
$schema->dropColumn('new_email');
$schema->save();
```

To drop table call `declareDropped` method of your schema prior to save:

```php
$schema->declareDropped();
$schema->save();
```

## Clean Table schema
In some cases you might want table schema strictly follow declared elements and automatically delete all non declared columns:

```php
$schema->setState(null);
```

Now you are able to redefine table schema.

## Work with Comparator
To get access to table state comparator use `getComparator` method of your schema:

```php
dump($schema->getComparator()->addedColumns());
```

> You can use comparator to generate migrations instead of letting DBAL sync your schemas.

## Sync multiple Tables
In some cases you might want to create multiple linked tables. In order to handle such operation feed your table schemas
into `Spiral\Database\Schema\Reflector`:

```php
$schema = $database->table('table_a')->getSchema();
$schema->primary('id');

$schemaB = $database->table('table_b')->getSchema();
$schemaB->primary('id');
$schemaB->integer('a_id');
$schemaB->foreign('a_id')->references('table_a', 'id');

$r = new \Spiral\Database\Schema\Reflector();
$r->addTable($schemaB);
$r->addTable($schema);

$pool->run();
```

> `Spiral\Database\Schema\Reflector` will sort your tables based on their cross dependencies.
