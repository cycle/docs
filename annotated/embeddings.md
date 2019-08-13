# Embeddings
ORM can simplify the definition of large entities by proving the ability to split some of the columns into embedded entity. Embedded entities by default will always be loaded with parent object, however, partial entity selection is possible as well.

> Embedded entities does not support relations at the moment.

## Definition
To define embeddable entity use `@Embeddable` annotation. As with `@Entity` you are able to define custom mapper or associate additional columns/indexes using `@Table` annotation.

```php
/** @Embeddable */
class Address 
{
    /** @Column(type = "string") */ 
    public $country;
  
    /** @Column(type = "string") */ 
    public $city;
  
    /** @Column(type = "string") */ 
    public $address;
}
```

> You do not need to define `primary` column, this column will be inherited from parent entity. Mapper methods `queueDelete`, `queueCreate` and `queueUpdate` would never be invoked due the deletation to the parent mapper.

To embedd entity to another object use `@Embedd` annotation:

```php
/**  
 * @Entity
 */
class User 
{
    /** @Column(type = "primary") */
    public $id;
    
    /** @Embedd(target = "Address") */
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
By default, all embedded entity columns will be stored in owning entity table without any prefix, you can define custom prefix using
`columnPrefix` option of `@Embeddable` annotation:

```php
/** @Embeddable(columnPrefix = "address_") */
class Address 
{
    /** @Column(type = "string") */ 
    public $country;
  
    /** @Column(type = "string") */ 
    public $city;
  
    /** @Column(type = "string") */ 
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
    /** @Column(type = "primary") */
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

## Query Embedded entity separatelly
It is possible to query embedded entity separatelly from parent, hovewer, you must clearly know the `role` of such entity as class name is forbidden (in order to allow usage of the emedding inside different parents). Usually such role will be composed using parent and entity role (and ":" separator).

```php
$orm->getRepository("user:address")->findAll();
```

> Make sure you know what you doing.
