# Morphed Relations
In some cases you might want to associate your entity with any of external entities which support given type. This can be achieved
using polymorphed relations associated to entity interface. Such approach supporded for `hasOne`, `hasMany` and `belongsTo` relations.

Relations points to external entity using complex key [role, outerKey].

> Avoid use of theese relations when possible.

## Definition
In order to associate entity with one of multiple external objects you have to define common interface first:

```php
/** @entity */
class User implements ImageHolderInterface
{
    // ...
}

/** @entity */
class Post implements ImageHolderInterface
{
    // ...
}
```

Now we can declare our entity to point to one of given entities:

```php
/** @entity */
class Image 
{
    // ...
    
    /** @belongsToMorphed(target = "ImageHolderInterface") */
    public $imageHolder;
}
```

You can use your relation as standard hasOne after that. **Note, eager loading is not possible with `belongsToMorphed` relation type.**

## Variations
ORM provide three basic relations for polymorphic connections:

### BelongsToMorphed
Use cases: image attached to (post, user, comment). Relation is similar to `belongsTo` but does not support eager loading, FKs or select quering. The relation must point to entity interface.

```php
/** @entity */
class Image 
{
    // ...
    
    /** @belongsToMorphed(target = "ImageHolderInterface") */
    public $imageHolder;
}
```

Relation options include:

Option      | Value  | Comment
---         | ---    | ----
cascade     | bool   | Automatically save related data with source entity, defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent), defaults to `true`
innerKey    | string | Inner key in source entity, defaults to `{relationName}_{outerKey}`
outerKey    | string | Outer key in the related entity, by default primary key
morphKey    | string | Name of key to store related entity role (by default `{relationName}_role`
morphKeyLength | int | The lenght of morph key, defaults to 32
indexCreate | bool   | Create an index on morphKey and innerKey, defaults to `true`

### MorphedHasOne
Declared the ability to own the entity from multiple entity types (example user/post/comment has image). The relation must point to entity role or class.

```php
/** @entity */
class User 
{
    // ...
    
    /** @morphedhasOne(target = "Image") */
    public $image;
}
```

Additional options include:

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach (default `lazy`)
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}` 
morphKey    | string | Name of key to store related entity role (by default `{relationName}_role`
morphKeyLength | int | The lenght of morph key, defaults to 32
indexCreate | bool   | Create an index on morphKey and innerKey, defaults to `true`

> As in case with `belongsToMorphed` FKs are not supported. You can query or eager load this relation as any other relation types.

### MorphedHasMany
Declared the ability to own the entity from multiple entity types (example post/article has comments). The relation must point to entity role or class.

```php
/** @entity */
class User 
{
    // ...
    
    /** @morphedhasMany(target = "Image") */
    public $images;
}
```

Additional options include:

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach (default `lazy`)
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if the relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity defaults to the primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}`
where       | array  | Additional where condition to be applied for the relation, defaults to none.
morphKey    | string | Name of key to store related entity role (by default `{relationName}_role`
morphKeyLength | int | The lenght of morph key, defaults to 32
indexCreate | bool   | Create an index on morphKey and innerKey, defaults to `true`

> As in case with `belongsToMorphed` FKs are not supported. You can query or eager load this relation as any other relation types.
