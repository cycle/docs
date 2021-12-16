# Embedded Entities

The ORM can simplify the definition of large entities by providing the ability to split some of the columns into an
embedded entity. Embedded entities by default will always be loaded with the parent object. However, partial entity
selection is possible as well.

You can embed one or multiple entities inside another object using the parent object table as a data source. It can be
achieved using the `embedded` relation type and might be useful to perform de-composition of your entity. Such a 
relation also allows lazy and eager (default) loading of embedded entities, or the ability to retrieve entities 
separately (without loading parent model).

> Embedded entities do not support relations at the moment.

## Definition

To define an embeddable entity use the `#[Embeddable]` attribute. As with `#[Entity]`, you are able to define a custom
mapper or associate additional columns/indexes using the `#[Table]` attribute.

```php
use Cycle\Annotated\Annotation\Embeddable;
use Cycle\Annotated\Annotation\Column;

#[Embeddable]
class UserCredentials
{
    #[Column(type: 'string(255)')]
    public string $username;

    #[Column(type: 'string')]
    public string $password;
}
```

> You do not need to define the `primary` column, this column will be inherited from the parent entity. Mapper
> methods `queueDelete`, `queueCreate` and `queueUpdate` would never be invoked due to the delegation to the parent
> mapper.

To embed an entity to another object use the `#[Embedded]` attribute:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[Embedded(target: 'UserCredentials')]
    public UserCredentials $credentials;

    public function __construct()
    {
        $this->credentials = new UserCredentials();
    }
}
```

> Make sure to init your relation to be able to use a newly created model.

Read more about embeddings [here](/docs/en/annotated/embeddings.md).

Embedded relations support the following options:

Option      | Value  | Comment
---         | ---    | ----
load        | lazy/eager | Relation load approach. Defaults to `eager`)

## Column Mapping

By default, all embedded entity columns will be stored in the owning entity table without any prefix. If desired, you
can define a custom prefix using the `columnPrefix` option of the `#[Embeddable]` attribute:


```php
use Cycle\Annotated\Annotation\Embeddable;
use Cycle\Annotated\Annotation\Column;

#[Embeddable(columnPrefix: 'credentials_')]
class UserCredentials
{
    #[Column(type: 'string(255)')]
    public string $username;

    #[Column(type: 'string')]
    public string $password;
}
```

## Usage

You can use the relation right after a schema update (embedded columns will be added to parent entity table):

```php
$u = new User();
$u->credentials->username = 'username';
$u->credentials->password = 'password';

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u);
$manager->run();
```

### Querying

You can query embedded entities as your would do for any other relation:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('credentials.username', 'john.smith');
```

## Eager and Lazy Loading

By default, all embedded entities will be loaded with the parent object. To alter this behavior use the `load` option of
the `#[Embedded(...)]` relation annotation:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public int $id;

    #[Embedded(target: UserCredentials::class, load: 'lazy')]
    public UserCredentials $credentials;
}
```

Now, in order to preload embedded entity you have to explicitly use the `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('credentials.username', 'username');

print_r($select->load('credentials')->fetchAll());
```

## Query Embedded entity separately

It is possible to query the embedded entity separately from the parent, however, you must clearly know the `role` of
such entity as using the class name is forbidden (in order to allow usage of the embedding inside different parents).
Usually, such role will be composed using parent and entity role with ":" separator.

```php
$orm->getRepository("user:credentials")->findAll();
```

> Make sure you know what you are doing.
