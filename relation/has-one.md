# Has One
The Has One relation defines that entity exclusively owns another entity in a form of parent-children. Consider this relation as form of decomposition with ability to store data in external table.

## Definition
To define Has One relation using annotated enties extension use:

```php
/** @entity */ 
class User 
{
    // ...
    
    /** @hasOne(target = "Address") */
    protected $address;
}
```

> You must properly handle the cases when relation is not initialized (`null`)!

By default, ORM will generate outer key in relation object using parent entity role and inner key (primary key by default) values. As result column and FK will be added to Address entity on `user_id` column.

Option      | Value  | Comment
---         | ---    | ----
load        | lazy|eager | Relation load approach (default `lazy`)
cascade     | bool   | Automatically save related data with parent entity, defaults to `true`
nullable    | bool   | Defines if relation can be nullable (child can have no parent), defaults to `false`
innerKey    | string | Inner key in parent entity, defaults to primary key
outerKey    | string | Outer key name, defaults to `{parentRole}_{innerKey}` 
fkCreate    | bool   | Set to true to automatically create FK on outerKey, defauls to `true`
fkAction    | CASCADE, NO ACTION, SET NULL | FK onDelete and onUpdate action, defaults to `CASCADE`  
indexCreate | bool   | Create index on outerKey, defaults to `true`

## Usage
To attach child object to the parent entity simple set the value on designated property:

```php
$u = new User();

// or setAddress() if you have a setter
$u->address = new Address();
```

The related object can be immediate saved into the database by persisting parent entity:

```php
$t = new Transaction($orm);
$t->persist($u);
$t->run();
```

To delete previously associated object simple set the property value to `null`:

```php
$u->setAddress(null);
```

The child object will be removed during the persist operation.

> To avoid child object removal (detach) set `nullable` true. In this case child outer key will be reset to `null`.

## Loading
To access related data simply call the method `load` of your `User` `Select` object:

```php
$u = $orm->getRepository(User::class)->select()->load('address')->wherePK(1)->fetchOne();
print_r($u->getAddress());
```

## Filtering
You can filter entity selection using related data, call method `with` of your entity `Select` to join related entity table:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->with('address')->where('address.city', 'New York')
    ->fetchAll();
    
print_r($users);
```

Cycle `Select` can automatically join related table on first `where` condition, previous example can be rewritten:

```php
$users = $orm->getRepository(User::class)
    ->select()
    ->where('address.city', 'New York')
    ->fetchAll();
    
print_r($users);
```

## Transfer Children
You can transfer related entity between two parents:

```php
$u1 = $orm->getRespository(User::class)->select()->load('address')->wherePK(1)->fetchOne();

$u2 = new User();
$u2->setAddress($u->getAddress());
$u1->setAddress(null);

$t = new Transaction($orm);
$t->persist($u1);
$t->persist($u2);
$t->run();
```
