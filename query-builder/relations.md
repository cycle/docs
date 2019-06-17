# Querying Relations
It is possible to use columns and values of entity relations while composing the query. Relation properties (columns) can be accessed
using dot notation `relation.property`. Please note, you must use domain specific property names, column names will be mapped automatically.

## Simple Condition
To query entity with constain applied to it's related entity:

```php
// find all users with published posts
$select->distinct()->where('posts.published', true);
```

Please note, all queried relations will be joined to the entity query, do not forget to add `distict` option to your query while joining
`hasMany`, `manyToMany` relations.

## Nested relations
It is possible to filter entity using any level of relations. For example:

```php
// find all users with posts which have approved comments
$select->distinct()->where('posts.comments.approved', true);
```

> Please note that deepening queries will affect your performance.

## Sorting and pagination
You can use method `orderBy` in combination with related entity column:

```php
$select->orderBy('posts.id', 'DESC');
```

> Please note, that usage of `limit` and `offset` methods only recommended in combination with `distinct`.
