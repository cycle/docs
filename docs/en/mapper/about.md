# Mappers

Every entity in Cycle ORM must have an associated Mapper object. Mappers are responsible for entity creation and
hydration, and they must issue a set of persist commands which will be processed later within the transaction.

## Interface

You can implement your own Mapper
using [Cycle/ORM/MapperInterface](https://github.com/cycle/orm/blob/2.x/src/MapperInterface.php):

The ORM will create a mapper using `Spiral\Core\FactoryInterface` which means your Mapper is able to request
dependencies available in the container associated with ORM Factory.

Some parameters will be provided by ORM itself, such as:

* **role** - entity role
* **schema** - entity schema
* **orm** - orm instance

You are able to use a single Mapper implementation for multiple entities.

## Available mappers

There are different types of mappers out of the box:

- [Proxy Mapper](/docs/en/mapper/proxy-mapper.md) - Provides the ability to carry data over the specific class instances
  using proxy classes.
- [Std Mapper](/docs/en/mapper/std-mapper.md) - Provides the ability to carry data over the StdClass objects.
- [Classless Mapper](/docs/en/mapper/classless-mapper.md) - Provides the ability to create proxy classes on the fly.
- [Promise Mapper](/docs/en/mapper/promise-mapper.md) - Provides the ability to carry data over the specific class
  instances with relation promises.

## Database Mapper

Database Mapper is an abstract mapper, that provides basic capabilities to work with entities persisted in SQL
databases.

You can implement your own mapper to implement custom entity carrying model but still rely on common SQL functionality.
Let's define our model first:

```php
class Entity
{
    public function __construct(
        private array $data = []
    ) {
    }

    public function setData(array $data): void
    {
        $this->data = $data;
    }

    public function getData(): array
    {
        return $this->data;
    }

    public function __get($name)
    {
        return $this->data[$name] ?? null;
    }

    // ...
}
```

We can now implement our mapper to handle creation and hydration of our entity:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\ORMInterface;
use Cycle\ORM\Mapper\DatabaseMapper;

final class CustomMapper extends DatabaseMapper
{
    private string $class;

    public function __construct(ORMInterface $orm, string $role)
    {
        parent::__construct($orm, $role);

        // entity class
        $this->class = $orm->getSchema()->define($role, Schema::ENTITY);
    }

    /** @inheritdoc */
    public function init(array $data): object
    {
        $class = $this->class;
        return new $class($data);
    }

    /** @inheritdoc */
    public function hydrate(object $entity, array $data): object
    {
        $entity->setData($data);
        return $entity;
    }

    /** @inheritdoc */
    public function extract(object $entity): array
    {
        return $entity->getData();
    }

    /**
     * Get entity columns.
     */
    public function fetchFields(object $entity): array
    {
        // fetch entity fields and ignore custom columns
        return array_intersect_key(
            $this->extract($entity),
            $this->columns + $this->parentColumns
        );
    }
    
    public function fetchRelations(object $entity): array
    {
        return array_intersect_key(
            $this->extract($entity),
            $this->relationMap->getRelations()
        );
    }
}
```

You can now create your entity and associate it with the custom mapper:

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity(
    mapper: CustomMapper::class
)]
#[Column(type: 'string', name: 'id', property: 'id')]
class User extends Entity
{

}
```

Update your ORM schema to register the entity. You can use your entity freely after this operation.

> ORM can work with different types of entities within one system.

## ActiveRecord

A similar approach can be used to implement AR-like entities. You would have to expose a global instance of ORM in order
to gain access to it from your entity's `save()` and `delete()` methods:

```php
use Cycle\ORM\EntityManager;

class Entity
{
    // ...

    public function save(): void
    {
        (new EntityManager(App::getORM()))->persist($this)->run();
    }

    public function delete(): void
    {
        (new EntityManager(App::getORM()))->delete($this)->run();
    }
}
```
