# Single Table Inheritance
ORM provides the ability to store multiple model variations inside one table. In order to achieve that you must extend your parent entity
and declare relations/columns specific to the child.

## Definition

```php
/** @entity */
class Post 
{
    /** @column(type=primary) */
    public $id;
}

/** @entity */
class Article extends Post
{
    /** @column(type=string) */
    public $articleTitle;
}
```

You can store `Article` same way as parent entity.

> Note, ORM will create special column in your entity table `_type` in which child id will be stored.

## Querying
You have to remember that fetching entities from repository might return any of child entity:

```php
// posts and articles
$posts = $orm->getRepository(Post:class)->findAll();
```

You are currently not allowed to assign custom repositories or constrains to child entity.
