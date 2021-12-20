# Embeddings

The ORM can simplify the definition of large entities by providing the ability to split some of the columns into an
embedded entity. Embedded entities by default will always be loaded with the parent object. However, partial entity
selection is possible as well.

> Embedded entities do not support relations at the moment.

## Definition

To define an embeddable entity use the `#[Embeddable]` attribute. As with `#[Entity]`, you are able to define a custom
mapper or associate additional columns/indexes using the `#[Table]` attribute.

```php
use Cycle\Annotated\Annotation\Embeddable;
use Cycle\Annotated\Annotation\Column;

#[Embeddable]
class Address
{
    #[Column(type: 'string')]
    public string $country;

    #[Column(type: 'string(32)')]
    public string $city;

    #[Column(type: 'string(100)')]
    public string $address;
}
```

> You do not need to define the `primary` column, this column will be inherited from the parent entity. Mapper 
> methods `queueDelete`, `queueCreate` and `queueUpdate` would never be invoked due to the delegation to the parent
> mapper.

To embed an entity to another object use the `#[Embedded]` attribute:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[Embedded(target: Address::class)]
    public Address $address;
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

By default, all embedded entity columns will be stored in the owning entity table without any prefix. If desired, you
can define a custom prefix using the `columnPrefix` option of the `#[Embeddable]` attribute:

```php
use Cycle\Annotated\Annotation\Embeddable;
use Cycle\Annotated\Annotation\Column;

#[Embeddable(columnPrefix: 'address_')]
class Address
{
    #[Column(type: 'string')]
    public string $country;

    #[Column(type: 'string')]
    public string $city;

    #[Column(type: 'string')]
    public string $address;
}
```

## Querying

You can query an embedded entity like any other relations:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');
```

## Eager and Lazy Loading

By default, all embedded entities will be loaded with the parent object. To alter this behavior use the `load` option
of `#[Embedded]` relation attribute:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[Embedded(target: Address:class, load: 'lazy')]
    public Address $address;
}
```

Now, in order to pre-load the embedded entity you have to explicitly use `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('address.country', 'USA');

print_r($select->load('address')->fetchAll());
```

## Query Embedded entity separately

It is possible to query the embedded entity separately from the parent, however, you must clearly know the `role` of
such entity as using the class name is forbidden (in order to allow usage of the embedding inside different parents).
Usually, such role will be composed using parent and entity role with ":" separator.

```php
$orm->getRepository("user:address")->findAll();
```

> Make sure you know what you are doing.
