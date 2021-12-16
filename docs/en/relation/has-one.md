# Has One:

The Has One relation defines that an entity exclusively owns another entity in a form of parent-child. Consider this
relation as a form of decomposition with the ability to store data in external table.

The HasOne relation is used to define the relation to one child object. This object will be automatically saved with its
parent (unless `cascade` option set to `false`). The simplest form of relation definition

## Definition

To define a Has One relation using the annotated entities' extension, use:

```php
use Cycle\Annotated\Annotation\Relation\HasOne;
use Cycle\Annotated\Annotation\Entity;

#[Entity]
class User
{
    // ...

    #[HasOne(target: Address::class)]
    public ?Address $address;
}
```

> You must properly handle the cases when the relation is not initialized (`null`)!

By default, ORM will generate an outer key in relation object using the parent entity's role and inner key (primary key
by default) values. As result column and FK will be added to Address entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach. Defaults to `lazy`
cascade     | bool   | Automatically save related data with parent entity. Defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent). Defaults to `false`
innerKey    | string | Inner key in parent entity. Defaults to primary key
outerKey    | string | Outer key name. Defaults to `{parentRole}_{innerKey}`
fkCreate    | bool   | Set to true to automatically create FK on outerKey. Defaults to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action. Defaults to `CASCADE`
fkOnDelete  | CASCADE, NO ACTION, SET NULL | FK onDelete action. It has higher priority than {$fkAction}. Defaults to @see {$fkAction}
indexCreate | bool   | Create index on outerKey. Defaults to `true`

## Usage

To attach the child object to the parent entity simple set the value on the designated property:

```php
$user = new User();

// or setAddress() method if you have a setter
$user->address = new Address();
```

The related object can be immediately saved into the database by persisting the parent entity:

```php
$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($user);
$manager->run();
```

To delete a previously associated object simply set the property value to `null`:

```php
$user->setAddress(null);
```

The child object will be removed during the persist operation.

> To avoid child object removal (detach) set `nullable` true. In this case, child outer key will be reset to `null`.

### Loading

To access related data simply call the method `load` of your `User`'s `Select` object:

```php
$user = $orm->getRepository(User::class)
    ->select()
    ->load('address')
    ->wherePK(1)
    ->fetchOne();

print_r($user->getAddress());
```

### Filtering

You can filter entity selection using related data, call the method `with` of your entity's `Select` to join the related
entity table:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('address')->where('address.city', 'New York')
    ->fetchAll();

print_r($users);
```

Cycle ORM `Select` can automatically join related tables on the first `where` condition. The previous example can be
rewritten:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->where('address.city', 'New York')
    ->fetchAll();

print_r($users);
```

### Transfer Children

You can transfer a related entity between two parents:

```php
$u1 = $orm->getRespository(User::class)->select()->load('address')->wherePK(1)->fetchOne();

$u2 = new User();
$u2->setAddress($u->getAddress());
$u1->setAddress(null);

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u1);
$manager->persist($u2);
$manager->run();
```
