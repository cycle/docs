# Embeddings
The ORM can simplify the definition of large entities by providing the ability to split some of the columns into an embedded entity. Embedded entities by default will always be loaded with the parent object. However, partial entity selection is possible as well.

> Embedded entities do not support relations at the moment.

## Definition
To define an embeddable entity use the `@Embeddable` annotation. As with `@Entity`, you are able to define a custom mapper or associate additional columns/indexes using the `@Table` annotation.

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

> You do not need to define the `primary` column, this column will be inherited from the parent entity. Mapper methods `queueDelete`, `queueCreate` and `queueUpdate` would never be invoked due to the delegation to the parent mapper.

To embed an entity to another object use the `@Embedded` annotation:

```php
/**
 * @Entity
 */
class User
{
    /** @Column(type = "primary") */
    public $id;

    /** @Embedded(target = "Address") */
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
By default, all embedded entity columns will be stored in the owning entity table without any prefix.
If desired, you can define a custom prefix using the `columnPrefix` option of the `@Embeddable` annotation:

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

## Querying
You can query an embedded entity as your would do for any other relations:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');
```

## Eager and Lazy Loading
By default, all embedded entities will be loaded with the parent object. To alter this behavior use the `load` option of `@Embedded` relation annotation:

```php
/** @Entity */
class User
{
    /** @Column(type = "primary") */
    public $id;

    /** @Embedded(target = "Address", load = "lazy") */
    public $address;
}
```

Now, in order to pre-load the embedded entity you have to explicitly use `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');

print_r($select->load('address')->fetchAll());
```

## Query Embedded entity separately
It is possible to query the embedded entity separately from the parent, however, you must clearly know the `role` of such entity as using the class name is forbidden (in order to allow usage of the embedding inside different parents). Usually, such role will be composed using parent and entity role with ":" separator.

```php
$orm->getRepository("user:address")->findAll();
```

> Make sure you know what you are doing.
