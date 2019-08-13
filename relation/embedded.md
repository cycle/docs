# Embedded Entities
You can embed one or multiple entities inside another object using parent object table as data source. It can be achieved using
`embedd` relation type and might be helpful to perform de-composition of your entity. Such relation also allows lazy and eager (default)
loading of embedded entities, or the ability to retrieve entities separally (without loading parent model).

## Definition
To define embedded entity using annotated extension you must declare your embedded entity first:

```php
/** @Embeddable */
class UserCredentials 
{
    /** @Column(type="string(255)") */
    public $username;
    
    /** @Column(type="string") */
    public $password;
}
```

> You do not need to declare primary key.

Now you can declare the usage of such entity if your model using relation of type `embedd`:

```php
/** @Entity */
class User 
{
    /** @Column(type = "primary") */
    public $id;
    
    /** @Embedd(target = "UserCredentials") */
    public $credentials;
    
    public function __construct()
    {
        $this->credentials = new UserCredentials();
    }
}
```

> Make sure to init your relation to be able to use newly created model.

Read more about embeddings [here](/annotated/embeddings.md).

Embedded relation support following options:

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach (default `eager`)

## Usage
You can use newly relation right after schema update (emebedded columns will be added to parent entity table):

```php
$u = new User();
$u->crendetials->username = 'username';
$u->crendetials->password = 'password';

$t = new Transaction($orm);
$t->persist($u);
$t->run();
```


## Quering
You can query emebedded entity as your would do for any other relations:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');
```

## Eager and Lazy Loading
By default, all embedded entities will be loaded with parent object. To alter this behaviour use `load` option of `@Embedd` relation annotation:

```php
/** @Entity */
class User 
{
    /** @Columntype = "primary") */
    public $id;
    
    /** @Embedd(target = "Address", load = "lazy") */
    public $address;
}
```

Now, in order to pre-load embedded entity you have to explicitly use `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');

print_r($select->load('address')->fetchAll());
```
