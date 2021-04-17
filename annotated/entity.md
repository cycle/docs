# Annotated Entities
The annotated entities extension is capable of indexing any domain entity in your project. To indicate that the class must be treated as a domain entity make sure to add the `@Entity` annotation to the DocComment.

```php
use Cycle\Annotated\Annotation\Entity;

/** @Entity */
class User
{
}
```

> Annotations are based on the Doctrine/Lexer package and support syntax similar to the Doctrine one.

## Entity
Usually, the single annotation `@Entity` is enough to describe your model. In this case, Cycle will automatically assign the generated
table name and role based on the class name. In the case of `User` the role will be `user`, database `null` (default) and table `users`.

You can tweak all of these values by setting `entity` options:

```php
use Cycle\Annotated\Annotation\Entity;

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

Some options can be used to overwrite default entity behaviour, for example to assign a custom entity repository:

```php
use Cycle\Annotated\Annotation\Entity;

/**
 * @Entity(repository = "Repository\UserRepository")
 */
class User
{
}
```

> Cycle can locate repository class names automatically, using current entity namespace as the base path.

Following entity options are available for customization:

Option | Value | Comment
--- | --- | ---
role           | string | Entity role. Defaults to the lowercase class name without a namespace
mapper         | class  | Mapper class name. Defaults to `Cycle\ORM\Mapper\Mapper`
repository     | class  | Repository class to represent read operations for an entity. Defaults to `Cycle\ORM\Select\Repository`
table          | string | Entity source table. Defaults to plural form of entity role
database       | string | Database name. Defaults to `null` (default database)
readonlySchema | bool   | Set to `true` to disable schema synchronization for the assigned table. Defaults to `false`
source         | class  | Entity source class (internal). Defaults to `Cycle\ORM\Select\Source`
constrain      | class  | Class name of constraint to be applied to every entity query. Defaults to `null`

For example, a  typical entity description might look like:

```php
use Cycle\Annotated\Annotation\Entity;

/**
 * @Entity(
 *    table      = "users",
 *    repository = "Repository\UserRepository",
 *    constrain  = "Constrain/SortByID"
 * )
 */
class User
{
}
```

## Columns
No entity can operate without some properties mapped to table columns. To map your property to the column add the annotation `@Column` to it. It's mandatory to specify the column type. You must always specify **one** primary (auto incremental) column for your entity.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

/** @Entity */
class User
{
    /** @Column(type = "primary") */
    protected $id;
}
```

> Read how to use non-incremental primary keys (for example UUID) in the Advanced section.

You can use multiple annotations at the same time:

```php
use Cycle\Annotated\Annotation as Cycle;

/** @Cycle\Entity */
class User
{
    /** @Cycle\Column(type = "primary") */
    protected $id;
}
```

> Annotation import is omitted in the following sections.

By default, the entity property will be mapped to the column with the same name as the property.
You can change it as follows:

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

Use the `default` option to specify the default value of the column:

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

While adding new columns to entities associated with non-empty tables you are required to either specify a default value or mark the column as nullable:

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
name | string | Column name. Defaults to the property name.
type | string | Column type with arguments.
primary | bool | Explicitly set column as primary key. Defaults to `false`
typecast | callable | Column typecast function. Defaults to one of (`int`\|`float`\|`bool`\|`datetime`) based on column type
nullable | bool | Set column as nullable. Defaults to `false`
default | mixed | Default column value. Defaults to `none`

Following column types are available:

Type        | Parameters                | Description
---         | ---                       | ---
**primary** | ---                       | Special column type, usually mapped as integer + auto-incrementing flag and added as table primary index column. You can define only one primary column in your table.
bigPrimary  | ---                       | Same as primary but uses bigInteger to store its values.
boolean     | ---                       | Boolean type, some databases will store it as integer (1/0).
integer     | ---                       | Database specific integer (usually 32 bits).
tinyInteger | ---                       | Small/tiny integer, check your DBMS to find size limitations.
bigInteger  | ---                       | Big/long integer (usually 64 bits), check your DBMS to find size limitations.
**string**  | [length:255]              | String with specified length, a perfect type for emails and usernames as it can be indexed.
text        | ---                       | Database specific type to store text data. Check DBMS to find size limitations.
tinyText    | ---                       | Tiny text, same as "text" for most of the databases. It differs only in MySQL.
longText    | ---                       | Long text, same as "text" for most of the databases. It differs only in MySQL.
double      | ---                       | [Double precision number.](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
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
The ORM supports the enum type for all available drivers. You must define enum options using comma separator:

```php
/** @Column(type = "enum(active,disabled)", default = "active") */
protected $status;
```

## Table Extension
In some cases you might want to specify additional table columns and indexes without the link to the entity properties. This can be achieved using `@Table` annotation:

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
The Annotated Entities extension supports the ability to merge table definitions provided by linked Mapper, Source, Repository and Constrain classes. This approach can be useful in cases when you want to implement domain functionality like auto timestamps or soft deletes.

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
