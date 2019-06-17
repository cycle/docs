# Embedded Entities
You can embedd one or multiple entities inside another object using parent object table as data source. It can be achieved using
`embedd` relation type and might be helpful to perform de-composition of your entity. Such relation also allows lazy and eager (default)
loading of embedded entities, or the ability to retrieve entities separally (without loading parent model).

## Definition
To define embedded entity using annotated extension you must declare your embedded entity first:

```php
/** @embeddable */
class UserCredentials 
{
    /** @column(type="string(255)") */
    public $username;
    
    /** @column(type=string) */
    public $password;
}
```

> You do not need to declare primary key.

Now you can declare the usage of such entity if your model using relation of type `embedd`:

```php
/** @entity */
class User 
{
    /** $column(type = primary) */
    public $id;
    
    /** @embedd(target = UserCredentials) */
    public $credentials;
    
    public function __construct()
    {
        $this->credentials = new UserCredentials();
    }
}
```

> Make sure to init your relation to be able to use newly created model.
