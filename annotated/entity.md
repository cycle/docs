# Annotated Entities
Annotated entities extension is capable to index any domain entity in your project. To indicate that class must be treated as domain 
entity make sure to add `@entity` annotation to the docComment.

```php
/**
 * @entity
 */
class User 
{
}
```

> Annotations are based on Doctrine/Lexer package and support syntax similar to the Doctrine one.

## Entity
Usually, the single annotation `@entity` is enought to describe your model. In this case Cycle will automatically assign generated
table name and role based on class name. In case of `User` the role will be `user`, database `null` (default) and table `users`.

You can tweak any of this values by setting `entity` options:

```php
/**
 * @entity (
 *     role     = "user", 
 *     database = "database", 
 *     table    = "user_table"
 * )
 */
class User 
{
}
```

> You must manually set `role` and `table` for your classes if you use models which share same name.

Some options can be used to overwrite default entity behaviour, for example to assign custom entity repository:

```php
/**
 * @entity (repository = "Repository/UserRepository")
 */
class User 
{
}
```

> Cycle can locate repository class name automatically, using current entity namespace as base path.

Following entity options are available for customization:

Option | Value | Comment 
--- | --- | ---
role           | string | Entity role, defaults to lowercases class name without namespace
mapper         | class  | Mapper class name, defaults to `Cycle\ORM\Mapper\Mapper`
repository     | class  | Repository class to represent read operations for entity, defaults to `Cycle\ORM\Select\Repository`
table          | string | Entity source table, defaults to plural form of entity role
database       | string | Database name, defaults to `null` (default database)
readonlySchema | bool   | Set to true to disable schema synchronization for assigned table.
source         | class  | Entity source class (internal), defaults to `Cycle\ORM\Select\Source`
constrain      | class  | Class name of contrain to be applied to every entity query, defaults to `null`

For example typical entity description might look like:

```php
/**
 * @entity (
 *    repository = "Repository/UserRepository",
 *    table      = "users",
 *    constrain  = "Constrain/SortByID"
 * )
 */
class User 
{
}
```

## Columns


## Table Extension


## Merging annotations
