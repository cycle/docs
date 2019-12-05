# Relations
The Cycle Annotated package provides multiple annotations designed to describe entity relations. Each relation must be associated with specific entity properties in order to work. In addition, most of the relation options (such as the name of inner, outer keys) will be generated automatically.

> You can read more about relation configuration and usage in later sections.

## Common Statement
Each relation must have a proper `target` option. The target must point to either the related entity `role`, or to the class name. You are able to specify class names in a fully qualified (Namespace\Class) or using current entity namespace as the base path. You can use both `/` and `\` namespace separators.

## HasOne
The HasOne relation is used to define the relation to one child object. This object will be automatically saved with its parent (unless `cascade` option set to `false`). The simplest form of relation definition:

```php
/** @Entity */
class User
{
    // ...

    /** @HasOne(target = "Address") */
    protected $address;
}
```

> You must properly handle the cases when the relation not initialized (`null`)!

By default, the ORM will generate the outer key in the relation object using the parent entity role and inner key (primary key by default) values. As a result the column and FK will be added to Address entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
indexCreate | bool   | Create index on outerKey. Defaults to `true`

## HasMany
The HasMany relation provides the ability to link multiple child objects to one entity (parent). The related entities will be stored in a ` Doctrine\Common\Collections\Collection` object (ArrayCollection). You must initiate an empty collection in your class constructor in order to properly work with newly created entities.


```php
/** @Entity */
class User
{
    // ...

    /** @HasMany(target = "Post") */
    protected $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }
}
```

Multiple options are available for the configuration:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to the primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
where       | array  | Additional where condition to be applied for the relation. Defaults to none.
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
indexCreate | bool   | Create an index on outerKey. Defaults to `true`

## BelongsTo
In order to link the entity to its parent object use the relation's `belongsTo`. Please note, a relation is `nullable` by default.

```php
/** @Entity */
class Post
{
    // ...

    /** @BelongsTo(target = "User") */
    protected $author;
}
```

Customizable options:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with source entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent). Defaults to `true`
innerKey    | string | Inner key in source entity. Defaults to `{relationName}_{outerKey}`
outerKey    | string | Outer key in the related entity. Defaults to its primary key
fkCreate    | bool   | Set to true to automatically create FK on innerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
indexCreate | bool   | Create an index on innerKey. Defaults to `true`

## RefersTo
The RefersTo relation is similar to the BelongsTo relation, but must be used to establish **multiple relations** to the same entity (or in case of a **cyclic** relation). The most common example is the ability to store the last post posted by the user.

```php
/** @Entity */
class User
{
    // ...

    /** @RefersTo(target = "Post") */
    protected $lastPost;

    /** @HasMany(target = "Post") */
    protected $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    public function addPost(Post $p)
    {
        $this->posts->add($p);
        $this->lastPost = $p;
    }
}
```

Options:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to the primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `SET NULL`
indexCreate | bool   | Create an index on outerKey. Defaults to `true`

> You must use the `refersTo` relation for cyclic dependencies.

## ManyToMany
A relation of type ManyToMany provides a more complex connection with the ability to use an intermediate entity for the connection. This relation must be represented using `Cycle\ORM\Relation\Pivoted\PivotedCollection`. The relation requires the  `though` option with similar rules as `target`.

```php
/** @Entity */
class User
{
    // ...

    /** @ManyToMany(target = "Tag", though = "UserTag") */
    protected $tags;

    public function __construct()
    {
        $this->tags = new PivotedCollection();
    }
}
```

The relation defines a number of options to control a set of associated keys and conditions:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (pivot entity can exist without the parent(s)). Defaults to `true`
innerKey    | string | Inner key name in source entity. Defaults to the primary key
outerKey    | string | Outer key name in target entity. Defaults to the primary key
thoughInnerKey | string | Key name connected to the innerKey of source entity. Defaults to `{sourceRole}_{innerKey}`
thoughOuterKey | string | Key name connected to the outerKey of a related entity. Defaults to `{targetRole}_{outerKey}`
thoughWhere | array | Where conditions applied to `though` entity
where       | array | Where conditions applied to the related entity
fkCreate    | bool   | Set to true to automatically create FK on thoughInnerKey and thoughOuterKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `SET NULL`
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey]. Defaults to `true`

> Note, the relation option `though` is a typo of `through`, it will remain in this package until next major release of `cycle/annotated`.

## Morphed Relations
Cycle ORM provides support for polymorphic relations. Given relations can be used to link an entity to multiple entity types and select the desired object in runtime. Relations must be assigned to the entity interface rather than a specific role or class name.

```php
/** @Entity */
class User implements ImageHolderInterface
{
     // ...
}
```

BelongsToMorphed relations allows an entity to belong to any of the parents which implement given interface:

```php
/** @Entity */
class Image
{
    // ...

    /** @BelongsToMorphed(target = "ImageHolderInterface")*/
    protected $parent;
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if relation can be nullable. Defaults to `true`
innerKey    | string | Inner key name in source entity. Defaults to `{relation}_{innerKey}`
outerKey    | string | Outer key name in target entity. Defaults to primary key
morphKey | string | Contains target entity role name. Defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey. Defaults to 32
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey]. Defaults to `true`

> The index will be created on [outerKey, morphKey].

MorphedHasOne and MorphedHasMany is an inverse version of BelongsToMorphed.

```php
/** @Entity */
class User implements ImageHolderInterface
{
     // ...

     /** @MorphedHasOne(target = "Image")*/
     protected $image;
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if relation can be nullable. Defaults to `false`
innerKey    | string | Inner key name in source entity. Defaults to primary key
outerKey    | string | Outer key name in target entity. Defaults to `{relation}_{innerKey}`
morphKey    | string | Contains target entity role name. Defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey. Defaults to 32
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey]. Defaults to `true`

```php
/** @Entity */
class User implements ImageHolderInterface
{
     // ...

     /** @MorphedHasMany(target = "Image")*/
     protected $images;

     public function __construct()
     {
         $this->images = new ArrayCollection();
     }
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if the relation can be nullable. Defaults to `false`
innerKey    | string | Inner key name in source entity. Defaults to the primary key
outerKey    | string | Outer key name in target entity. Defaults to `{relation}_{innerKey}`
morphKey | string | Contains target entity role name. Defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey. Defaults to 32
where       | array | Where conditions applied to the related entity
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey]. Defaults to `true`

Please note, given relations would not be able to automatically create FK keys since the ORM is unable to decide which key must be used. Also, eager loading abilities are limited for such relations (join is only possible for `morphedHas*` relations).

## Inversing Relations
In some cases you might want to create an inversed relation automatically. Please note, you still have to create a property in order to store the related data (and initialize it in case of `many` relations).

To inverse a relation, you must use the option `inverse` with specified inversed relation name and type.

```php
/** @Entity */
class Post
{
    // ...

    /** @BelongsTo(target = "User", inverse = @Inverse(as = "posts", type = "hasMany")) */
    protected $user;
}
```
