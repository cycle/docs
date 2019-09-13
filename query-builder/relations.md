# Querying Relations
It is possible to use columns and values of entity relations while composing the query. Relation properties (columns) can be accessed
using dot notation `relation.property`. Please note, you must use domain-specific property names, column names will be mapped automatically.

## Simple Condition
To query entity with constain applied to it's related entity:

```php
// find all users with published posts
$select->distinct()->where('posts.published', true);
```

Please note, all queried relations will be joined to the entity query, do not forget to add the `distinct` option to your query while joining
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

## Loading Relations
Cycle ORM can pre-load most of relation types via `load` method called on select:

```php
$select->load('posts');
```

The load method will automatically pick the appropriate way to load related data (either using JOIN or WHERE IN approaches). You can also
use this method to load any relation level via dot notation:

```php
$select->load('posts.comments.author');
```

## Load with applied condition
You can load some relations partially (hasMany, manyToMany) and use DB level filtering by applying `where' option:

```php
$select->load('posts', [
    'where' => ['published' => true]
]);
```

Select separate filtered and loaded entities you can use `with` and `load` methods at the same time.

```php
$select->load('posts', [
    'where' => ['published' => true]
])
->with('posts')->where('posts.flagged', true);
```

> Find all users with flagged posts and load all published posts.

In some cases, you can also combine joining and relation together (make sure you know what are you doing). You can do that by pointing the source table alias to the `load` method:

```php
$selec->with('posts',[
    'as'    => 'posts', 
    'where' => ['flagged' => true'])
])->load('posts', ['using' => 'posts']);
```

Such query will find all entities with flagged posts and load these posts within one query (make sure to set the DISTINCT). Note, this is NOT optimization technique.

> LIMIT, ORDER BY are currently not supported as fetch scope (no BC expected).

## Load with filter by nested relation
You can not only load the relation chain but also filter your branches by their relations. For example, we can load all users and all users posts which have comments.

```php
$users->load('posts', [
    'where' => ['comments.id' => ['!=' => null]]
]);
```

You can also use alternative notation with additional `distinct` flag:

```php
$users->load('posts', [
    'where' => function($qb) {
        $qb->distinct()->where('comments.id', '!=', null);
    }
]);
```

Following SQL will be generated:

```sql
SELECT
"user"."id" AS "c0", "user"."email" AS "c1", "user"."balance" AS "c2"
FROM "user" AS "user"
```

```sql
SELECT DISTINCT
"user_posts"."id" AS "c0", "user_posts"."user_id" AS "c1", "user_posts"."title" AS "c2"
FROM "post" AS "user_posts"
INNER JOIN "comment" AS "user_posts_comments"
    ON "user_posts_comments"."post_id" = "user_posts"."id"
WHERE "user_posts"."user_id" IN (1, 2) AND ("user_posts_comments"."id" IS NOT NULL)
```

Please note, the relation is joined using `INNER JOIN` by default. You can alter this behavior.

To find all users and load only posts without comments:

```php
$users->load('posts', [
    'where' => function (Select\QueryBuilder $qb) {
        $qb->distinct()->with('comments', [
            'method' => Select\JoinableLoader::LEFT_JOIN]
        )->where('comments.id', '=', null);
    }
]);
```

```sql
SELECT
"user"."id" AS "c0", "user"."email" AS "c1", "user"."balance" AS "c2"
FROM "user" AS "user"
```

```sql
SELECT DISTINCT
"user_posts"."id" AS "c0", "user_posts"."user_id" AS "c1", "user_posts"."title" AS "c2"
FROM "post" AS "user_posts"
LEFT JOIN "comment" AS "user_posts_comments"
    ON "user_posts_comments"."post_id" = "user_posts"."id"
WHERE "user_posts"."user_id" IN (1, 2) AND ("user_posts_comments"."id" IS NULL)
```

You can also specify the join method in primary select query. Let's try to find all users with posts without comments and load only posts with comments for these users. We would have to use `options` of our relation to specifying the filter:


```php
$users
    ->distinct()
    ->with('posts.comments', [
        'method' => Select\JoinableLoader::LEFT_JOIN,
    ])
    ->where('posts.comments.id', null)
    ->load('posts', [
        'where' => function (Select\QueryBuilder $qb) {
            $qb->distinct()->where('comments.id', '!=', null);
        }
    ])->orderBy('user.id');
```

And generated SQLs:

```sql
SELECT DISTINCT
    "user"."id" AS "c0", "user"."email" AS "c1", "user"."balance" AS "c2"
FROM "user" AS "user"
INNER JOIN "post" AS "user_posts"
    ON "user_posts"."user_id" = "user"."id"
LEFT JOIN "comment" AS "user_posts_comments"
    ON "user_posts_comments"."post_id" = "user_posts"."id"
WHERE "user_posts_comments"."id" IS NULL
```

```sql
SELECT DISTINCT
    "user_posts"."id" AS "c0", "user_posts"."user_id" AS "c1", "user_posts"."title" AS "c2"
FROM "post" AS "user_posts"
INNER JOIN "comment" AS "user_posts_comments"
    ON "user_posts_comments"."post_id" = "user_posts"."id"
WHERE "user_posts"."user_id" IN (1, 2) AND ("user_posts_comments"."id" IS NOT NULL)
```
