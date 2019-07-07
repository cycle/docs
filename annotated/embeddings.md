# Embeddings
ORM can simplify the definition of large entities by proving the ability to split some of the columns into embedded entity. Embedded entities by default will always be loaded with parent object, however, partial entity selection is possible as well.

> Embedded entities does not support relations at the moment (and most likely will never will due to the conceptual limitation).

## Definition
To define embeddable entity use `@Embeddable` annotation. As with `@Entity` you are able to define custom mapper or associate additional columns/indexes using `@Table` annotation.

```php
/** @Embeddable */
class Address 
{
    /** @Column(type = string) */ 
    public $country;
  
    /** @Column(type = string) */ 
    public $city;
  
    /** @Column(type = string) */ 
    public $address;
}
```

> You do not need to define `primary` column, this column will be inherited from parent entity.

To embedd entity to another object use `@Embedd` annotation:

```php
/**  
 * @Entity
 */
class User 
{
    /** @Column(type = primary) */
    public $id;
    
    /** @Embedd(target = Address) */
    public $address;
}
```

Do not forget to initiate embedding in your entity:

```php
public function __construct()
{
    $this->address = new Address();
}
```

You can use embedding after the schema update:

```php
$user = new User();
$user->address->country = 'USA';
```

## Column Mapping
By default, all embedded entity column will be stored in owning entity table without any prefix, you can define custom prefix using
`columnPrefix` option of `@Embeddable` annotation:

```php
/** @Embeddable(columnPrefix = "address_") */
class Address 
{
    /** @Column(type = string) */ 
    public $country;
  
    /** @Column(type = string) */ 
    public $city;
  
    /** @Column(type = string) */ 
    public $address;
}
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
    /** @Column(type = primary) */
    public $id;
    
    /** @Embedd(target = Address, load = lazy) */
    public $address;
}
```

Now, in order to pre-load embedded entity you have to explicitly use `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');

print_r($select->load('address')->fetchAll());
```
