# Morphed Relations

Cycle ORM provides support for polymorphic relations. Given relations can be used to link an entity to multiple entity
types and select the desired object in runtime. Relations must be assigned to the entity interface rather than a
specific role or class name.

In some cases, you might want to associate your entity with any of the external entities which support the given type.
This can be achieved using polymorphed relations associated with the entity interface. Such an approach is supported for
the `hasOne`, `hasMany` and `belongsTo` relations.

Relations points to the external entity using a complex key [role, outerKey].

> [Avoid using](http://duhallowgreygeek.com/polymorphic-association-bad-sql-smell/) these relations when possible.

## Definition

In order to associate an entity with one of the multiple external objects you have to define a common interface first:

```php
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User implements ImageHolderInterface
{
    // ...
}

#[Entity]
class Post implements ImageHolderInterface
{
    // ...
}
```

Now we can declare our entity to point to one of the given entities:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed;

#[Entity]
class Image
{
    // ...

    #[BelongsToMorphed(taget: ImageHolderInterface::class)]
    public ImageHolderInterface $imageHolder;
}
```

You can use your relation as a standard hasOne after that. **Note, eager loading is not possible with `belongsToMorphed`
relation type.**

Note: In order to be able to use the annotation as `@BelongsToMorphed`, you need to import it
with `use Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed;`

## Variations

The ORM provides three basic relations for polymorphic connections:

### BelongsToMorphed

BelongsToMorphed relations allows an entity to belong to any of the parents which implement given interface:

Use cases: image attached to (post, user, comment). The relation is similar to `belongsTo` but does not support eager
loading, FKs or select querying. The relation must point to an entity interface.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\Morphed\BelongsToMorphed;

#[Entity]
class Image
{
    // ...

    #[BelongsToMorphed(taget: ImageHolderInterface::class)]
    public ImageHolderInterface $imageHolder;
}
```

Relation options include:

Option         | Value      | Comment
---            |------------| ----
load           | lazy/eager | Relation load approach. Defaults to `lazy`
cascade        | bool       | Automatically save related data with source entity. Defaults to `true`
nullable       | bool       | Defines if the relation can be nullable (child can have no parent). Defaults to `true`
innerKey       | string     | Inner key in source entity. Defaults to `{relationName}_{outerKey}`
outerKey       | string     | Outer key in the related entity. Defaults to primary key
morphKey       | string     | Name of key to store related entity role. Defaults to `{relationName}_role`
morphKeyLength | int        | The length of morph key. Defaults to 32
indexCreate    | bool       | Create an index on morphKey and innerKey. Defaults to `true`

### MorphedHasOne

MorphedHasOne is an inverse version of BelongsToMorphed.

You can own the entity from multiple entity types (example user/post/comment has an image). The relation must point to
an entity role or class.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasOne;

#[Entity]
class User
{
    // ...

    #[MorphedHasOne(target: Image::class)]
    public $image;
}
```

Additional options include:

Option         | Value      | Comment
---            |------------| ----
load           | lazy/eager | Relation load approach. Defaults to `lazy`
cascade        | bool       | Automatically save related data with parent entity. Defaults to `true`
nullable       | bool       | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey       | string     | Inner key in parent entity. Defaults to the primary key
outerKey       | string     | Outer key name. Defaults to `{parentRole}_{innerKey}`
morphKey       | string     | Name of key to store related entity role. Defaults to `{relationName}_role`
morphKeyLength | int        | The length of morph key. Defaults to 32
indexCreate    | bool       | Create an index on morphKey and innerKey. Defaults to `true`

> As in the case with `belongsToMorphed`, FKs are not supported. You can query or eager load this relation as any other relation types.

### MorphedHasMany

MorphedHasMany is an inverse version of BelongsToMorphed.

You can own the entity from multiple entity types (example post/article has comments). The relation must point to an
entity role or class.

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Relation\Morphed\MorphedHasMany;

#[Entity]
class User
{
    // ...

    #[MorphedHasMany(target: Image::class)]
    public $images;
}
```

Additional options include:

Option         | Value      | Comment
---            |------------| ----
load           | lazy/eager | Relation load approach. Defaults to `lazy`
cascade        | bool       | Automatically save related data with parent entity. Defaults to `true`
nullable       | bool       | Defines if the relation can be nullable (child can have no parent). Defaults to `false`
innerKey       | string     | Inner key in parent entity. Defaults to the primary key
outerKey       | string     | Outer key name. Defaults to `{parentRole}_{innerKey}`
where          | array      | Additional where condition to be applied for the relation. Defaults to none.
morphKey       | string     | Name of key to store related entity role. Defaults to `{relationName}_role`
morphKeyLength | int        | The length of morph key. Defaults to 32
indexCreate    | bool       | Create an index on morphKey and innerKey. Defaults to `true`
collection     | string     | Collection type that will contain loaded entities. By defaults uses `Cycle\ORM\Collection\ArrayCollectionFactory`

> As in case with `belongsToMorphed`, FKs are not supported. You can query or eager load this relation as any other
> relation types.

---

Please note, given relations would not be able to automatically create FK keys since the ORM is unable to decide which
key must be used. Also, eager loading abilities are limited for such relations (join is only possible for `morphedHas*`
relations).
