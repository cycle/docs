# Embedded Entities
You can embed one or multiple entities inside another object using the parent object table as a data source. It can be achieved using the
`embedded` relation type and might be useful to perform de-composition of your entity. Such a relation also allows lazy and eager (default)
loading of embedded entities, or the ability to retrieve entities separately (without loading parent model).

## Definition
To define embedded entity using the annotated extension, you must first declare your embedded entity:

```php
use Cycle\Annotated\Annotation\Embeddable;
use Cycle\Annotated\Annotation\Column;

#[Embeddable]
class UserCredentials
{
    #[Column(type: 'string(255)')]
    public $username;

    #[Column(type: 'string')]
    public $password;
}
```

> You do not need to declare a primary key.

Now you can declare the usage of this entity if your model using the relation of type `embedd`:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public $id;

    #[Embedded(target: 'UserCredentials')]
    public $credentials;

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

## Usage
You can use the relation right after a schema update (embedded columns will be added to parent entity table):

```php
$u = new User();
$u->credentials->username = 'username';
$u->credentials->password = 'password';

$manager = new \Cycle\ORM\EntityManager($orm);
$manager->persist($u);
$state = $manager->run();
```


### Querying
You can query embedded entities as your would do for any other relation:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('credentials.username', 'username');
```

## Eager and Lazy Loading
By default, all embedded entities will be loaded with the parent object. To alter this behavior use the `load` option of the `@Embedd` relation annotation:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column
use Cycle\Annotated\Annotation\Relation\Embedded;

#[Entity]
class User
{
    #[Column(type: 'primary')]
    public $id;

    #[Embedded(target: 'UserCredentials', load: 'lazy')]
    public $credentials;
}
```

Now, in order to pre-load embedded entity you have to explicitly use the `load()` method of your select:

```php
$select = $orm->getRepository(User::class)->select();
$select->where('credentials.username', 'username');

print_r($select->load('credentials')->fetchAll());
```
