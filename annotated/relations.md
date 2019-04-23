# Relations
Cycle Annotated package provides multiple annotations designed to describe entity relations. Each relation must be associated with specific entity property in order to work. In addition, most of relation options (such as name of inner, outer keys) will be generated automatically.

> You can read more about relation configuration and usage in later sections.

## Common Statement
Each relation must have proper `target` option. Target must point to either related entity `role` or to class name. You are able to specify class names in a fully qualified for (Namespace\Class) or using current entity namespace as base path. You can use both `/` and `\` namespace separators.

## HasOne
HasOne relation used to define the relation to one child object. This object will be automatically saved with it's parent (unless `cascade` option set to `false`). The simplest form of relation definition:

```php
/** @entity */ 
class User 
{
    // ...
    
    /** @hasOne(target = "Address") */
    protected $address;
}
```

> You must properly handle the cases when relation not initialized (`null`)!

By default, ORM will generate outer key in relation object using parent entity role and inner key (primary key by default) values. As result column and FK will be added to Address entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}` 
fkCreate    | bool   | Set to true to automatically create FK on outerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `CASCADE`  
indexCreate | bool   | Create index on outerKey, defaults to `true`

## HasMany
HasMany relation provides ability to link multiple child objects to one entity (parent). The related entities will be stored in ` Doctrine\Common\Collections\Collection` object (ArrayCollection). You must initiate empty collection in your class constructor in order to properly work with newly created entities.


```php
/** @entity */ 
class User 
{
    // ...
    
    /** @hasMany(target = "Post") */
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
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}`
where       | array  | Additional where condition to be applied for the relation, defaults to none.
fkCreate    | bool   | Set to true to automatically create FK on outerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `CASCADE`  
indexCreate | bool   | Create index on outerKey, defaults to `true`

## BelongsTo
In order to link the entity to it's parent object use relation `belongsTo`. Please note, given relation is `nullable` by default.

```php
/** @entity */
class Post 
{
    // ...
    
    /** @belongsTo(target = "User") */
    protected $author;
}
```

Customizable options:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with source entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `true`
innerKey    | string | Inner key in source entity, defaults to `{relationName}_{outerKey}`
outerKey    | string | Outer key in related entity, by default primary key
fkCreate    | bool   | Set to true to automatically create FK on innerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `CASCADE`  
indexCreate | bool   | Create index on innerKey, defaults to `true`

## RefersTo
The RefersTo relation is similar to BelongsTo relation but must be used to establish **multiple relations** to the same entity (or in case of **cyclic** relation). The most common example is ability to store last post stored by users.

```php
/** @entity */ 
class User 
{
    // ...
 
    /** @refersTo(target = "Post") */
    protected $lastPost;
 
    /** @hasMany(target = "Post") */
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
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}`
where       | array  | Additional where condition to be applied for the relation, defaults to none.
fkCreate    | bool   | Set to true to automatically create FK on outerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `SET NULL`  
indexCreate | bool   | Create index on outerKey, defaults to `true`

> You must use `refersTo` relation for cyclic dependencies.

## ManyToMany
Relation of type ManyToMany provide more complex connection with ability to use intermediate entity for the connection. This relation must be represented using `Cycle\ORM\Relation\Pivoted\PivotedCollection`. The relation required definition of `though` option with similar rules as `target`.

```php
/** @entity */ 
class User 
{
    // ...

    /** @manyToMany(target = "Tag", though = "UserTag") */
    protected $tags;
    
    public function __construct()
    {
        $this->tags = new PivotedCollection();
    }
}
```

The relation defines number of options to control set of associated keys and conditions:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (pivot entity can exists without parent(s)), defaults to `true`
innerKey    | string | Inner key name in source entity, default to primary key
outerKey    | string | Outer key name in target entity, default to primary key
thoughInnerKey | string | Key name connected to the innerKey of source entity, defaults to `{sourceRole}_{innerKey}` 
thoughOuterKey | string | Key name connected to the outerKey of related entity, defaults to `{targetRole}_{outerKey}` 
thoughWhere | array | Where conditions applied to `though` entity
where       | array | Where conditions applied to related entity
fkCreate    | bool   | Set to true to automatically create FK on thoughInnerKey and thoughOuterKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `SET NULL`  
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`

## Morphed Relations
Cycle ORM provides support for the polimorphic relations. Given relations can be used to link entity to multiple entity types and seleted the desired object in runtime. Relations must be assigned to the entity interface rather than specific role or class name.

```php
/** @entity */
class User implements ImageHolderInterface 
{
     // ...
}
```

BelongsToMorphed relations allows entity to belong to any of the parents which implement given interface:

```php
/** @entity */
class Image
{
    // ...
    
    /** @belongsToMorphed(target = "ImageHolderInterface")*/
    protected $parent;
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable, defaults to `true`
innerKey    | string | Inner key name in source entity, default to `{relation}_{innerKey}`
outerKey    | string | Outer key name in target entity, default to primary key
morphKey | string | Contains target entity role name, defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey, defaults to 32
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`

> The index will be created on [outerKey, morphKey].

MorphedHasOne and MorphedHasMany is an inverse version of BelongsToMorphed.

```php
/** @entity */
class User implements ImageHolderInterface 
{
     // ...
     
     /** @morphedHasOne(target = "Image")*/
     protected $image;
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable, defaults to `false`
innerKey    | string | Inner key name in source entity, default to primary key
outerKey    | string | Outer key name in target entity, default to `{relation}_{innerKey}`
morphKey    | string | Contains target entity role name, defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey, defaults to 32
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`

```php
/** @entity */
class User implements ImageHolderInterface 
{
     // ...
     
     /** @morphedHasMany(target = "Image")*/
     protected $images;
     
     public function __construct() 
     {
         $this->images = new ArrayCollection();
     }
}
```

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable, defaults to `false`
innerKey    | string | Inner key name in source entity, default to primary key
outerKey    | string | Outer key name in target entity, default to `{relation}_{innerKey}`
morphKey | string | Contains target entity role name, defaults to `{relation}_role`
morphKeyLength | int | The lengths of the morphKey, defaults to 32
where       | array | Where conditions applied to related entity
indexCreate | bool   | Create index on [thoughInnerKey, thoughOuterKey], defaults to `true`

Please note, given relations would not be able to automatically cretate FK keys since ORM is unable to decide which key must be used. Also, eager loading abilities are limited for such relations (join is only possible for `morphedHas*` relations).

## Inversing Relations
In some cases you might want to create inversed relation automatically. Please note, you still have to create property in order to store the related data (and initialize it in case of `many` relations).

To inverse relation you must use option `inverse` with specified inversed relation name and type.

```php
/** @entity */ 
class Post 
{
    // ...
    
    /** @belongsTo(target = "User", inverse = @inverse(as = "posts", type = hasMany)) */
    protected $user;
}
```
