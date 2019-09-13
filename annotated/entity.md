# Annotated Entities
Annotated entities extension is capable of indexing any domain entity in your project. To indicate that class must be treated as a domain entity make sure to add `@Entity` annotation to the DocComment.

```php
/** @Entity */
class User 
{
}
```

> Annotations are based on Doctrine/Lexer package and support syntax similar to the Doctrine one.

## Entity
Usually, the single annotation `@Entity` is enough to describe your model. In this case, Cycle will automatically assign the generated
table name and role based on the class name. In the case of `User` the role will be `user`, database `null` (default) and table `users`.

You can tweak any of this values by setting `entity` options:

```php
/**
 * @Entity(
 *     role     = "user", 
 *     database = "database", 
 *     table    = "user_table"
 * )
 */
class User 
{
}
```

> You must manually set `role` and `table` for your classes if you use models that share the same name.

Some options can be used to overwrite default entity behaviour, for example to assign custom entity repository:

```php
/**
 * @Entity(repository = "Repository/UserRepository")
 */
class User 
{
}
```

> Cycle can locate repository class name automatically, using current entity namespace as the base path.

Following entity options are available for customization:

Option | Value | Comment 
--- | --- | ---
role           | string | Entity role defaults to lowercases class name without a namespace
mapper         | class  | Mapper class name defaults to `Cycle\ORM\Mapper\Mapper`
repository     | class  | Repository class to represent read operations for an entity defaults to `Cycle\ORM\Select\Repository`
table          | string | Entity source table defaults to plural form of entity role
database       | string | Database name, defaults to `null` (default database)
readonlySchema | bool   | Set to true to disable schema synchronization for the assigned table, defaults to `false`
source         | class  | Entity source class (internal), defaults to `Cycle\ORM\Select\Source`
constrain      | class  | Class name of constraint to be applied to every entity query, defaults to `null`

For example typical entity description might look like:

```php
/**
 * @Entity(
 *    table      = "users",
 *    repository = "Repository/UserRepository",
 *    constrain  = "Constrain/SortByID"
 * )
 */
class User 
{
}
```

## Columns
No entity can operate without some properties mapped to table columns. To map your property to the column add annotation `@Column` to it. It's mandatory to specify column type. You must always specify **one** primary (auto incremental) column for your entity.

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
}
```

> Read how to use non-incremental primary keys (for example UUID) in the Advanced section.

By default, the entity properly will be mapped to the column with the same name as the property, you can change it:

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
    
    /** @Column(type = "string", name = "username") */
    protected $login;
}
```

Some column types support additional arguments, such as length, values, etc.

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
    
    /** @Column(type = "string(32)") */
    protected $login;
    
    /** @Column(type = "enum(active,disabled)") */
    protected $status;
        
    /** @Column(type = "decimal(5,5)") */
    protected $balance;
}
```

Use `default` option to specify the default value of the column:

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
    
    /** @Column(type = "enum(active,disabled)", default = "active") */
    protected $status;
}
```

While adding new columns to the entities associated with non-empty tables you are required to either specify a default value or mark column as nullable:

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
    
    /** @Column(type = "string(64)", nullable = true) */
    protected $password;
}
```

Following options are available for configuration:

Option | Value | Comment
--- | --- | ---
name | string | Column name, default to the property name.
type | string | Column type with arguments.
primary | bool | Explicitly set column as primary key defaults to `false`
typecast | callable | Column typecast function, defaults to one of (int|float|bool|datetime) based on column type
nullable | bool | Set column as nullable, defaults to `false`
default | mixed | Default column value defaults to `none`

Following column types are available:

Type        | Parameters                | Description
---         | ---                       | ---
**primary** | ---                       | Special column type, usually mapped as integer + auto-incrementing flag and added as table primary index column. You can define only one primary column in your table (you still can create a compound primary key, see below).
bigPrimary  | ---                       | Same as primary but uses bigInteger to store its values.
boolean     | ---                       | Boolean type, some databases will store it as integer (1/0).
integer     | ---                       | Database specific integer (usually 32 bits).
tinyInteger | ---                       | Small/tiny integer, check your DBMS to check it's the size.
bigInteger  | ---                       | Big/long integer (usually 64 bits), check your DBMS to check it's the size.
**string**  | [length:255]              | String with specified length, a perfect type for emails and usernames as it can be indexed. 
text        | ---                       | Database specific type to store text data. Check DBMS to find size limitations.
tinyText    | ---                       | Tiny text, same as "text" for most of the databases. It differs only in MySQL.
longText    | ---                       | Long text, same as "text" for most of the databases. It differs only in MySQL.
double      | ---                       | [Double precision number.] (https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
float       | ---                       | Single precision number, usually mapped into "real" type in the database. 
decimal     | precision,&nbsp;[scale:0] | Number with specified precision and scale.
datetime    | ---                       | To store specific date and time, DBAL will automatically force UTC timezone for such columns.
date        | ---                       | To store date only, DBAL will automatically force UTC timezone for such columns.
time        | ---                       | To store time only.
*timestamp* | ---                       | Timestamp without a timezone, DBAL will automatically convert incoming values into UTC timezone. Do not use such column type in your objects to store time (use `datetime` instead) as timestamps will behave very specific to select DBMS.
binary      | ---                       | To store binary data. Check specific DBMS to find size limitations.
tinyBinary  | ---                       | Tiny binary, same as "binary" for most of the databases. It differs only in MySQL.
longBinary  | ---                       | Long binary, same as "binary" for most of the databases. It differs only in MySQL.
json        | ---                       | To store JSON structures, such type usually mapped to "text", only Postgres support it natively.

## Enums
ORM support enum type for all available drivers, you must define enum options using comma separator:

```php
/** @Column(type = "enum(active,disabled)", default = "active") */
protected $status;
```

## Table Extension
In some cases you might want to specificy additional table columns and indexes without the link to the entity properies. This can be achieved using `@Table` annotation:

```php
/**  
 * @Entity
 * @Table(
 *      columns={"created_at": @Column(type = "datetime"), "deleted_at": @Column(type = "datetime")},
 *      indexes={
 *             @Index(columns = {"username"}, unique = true), 
 *             @Index(columns = {"status"})
 *      }
 * )
 */
class User 
{
    /** @Column(type = "primary") */
    protected $id;
    
    /** @Column(type ="string(32)") */
    protected $username;
    
    /** @Column(type = "enum(active,disabled)", default = "active") */
    protected $status;
}
```

> The column definition is identical to the one used for the property.

## Merging annotations
Annotated Entities extension support ability to merge table definitions provided by linked Mapper, Source, Repository and Constrain classes. Such an approach can be useful in cases when you want to implement domain wise functionality like auto timestamps or soft deletes.

```php
/**
 * @Entity(repository = "Repository/UserRepository")
 */
class User 
{
}
```

You can also use short annotation declaration:

```php
/**
 * @Table(
 *     columns={"created_at": @Column("datetime")},
 *     indexes={@Index(columns = {"created_at"})}
 * ) 
 */
class UserRepository extends Repository
{

}
```
