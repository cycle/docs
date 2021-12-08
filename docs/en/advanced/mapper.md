# Mappers

Every entity in Cycle ORM must have an associated Mapper object. Mappers are responsible for entity creation and
hydration, and they must issue a set of persist commands which will be processed later within the transaction.

## Interface

You can implement your own Mapper using `Cycle/ORM/MapperInterface`:

```php
use Cycle\ORM\Command\CommandInterface;
use Cycle\ORM\Exception\MapperException;
use Cycle\ORM\Heap\Node;
use Cycle\ORM\Heap\State;

/**
 * Provides basic capabilities for CRUD operations with given entity class (role).
 */
interface MapperInterface
{
    /**
     * Get role name mapper is responsible for.
     */
    public function getRole(): string;

    /**
     * Init empty entity object. Returns empty entity.
     *
     * @param array $data Raw data. You shouldn't apply typecasting to it.
     */
    public function init(array $data, string $role = null): object;

    /**
     * Cast raw data to configured types.
     */
    public function cast(array $data): array;

    /**
     * Hydrate entity with dataset.
     *
     * @template T
     *
     * @param object<T> $entity
     * @param array $data Prepared (typecasted) data
     *
     * @throws MapperException
     *
     * @return T
     */
    public function hydrate(object $entity, array $data): object;

    /**
     * Extract all values from the entity.
     */
    public function extract(object $entity): array;

    /**
     * Get entity columns.
     */
    public function fetchFields(object $entity): array;

    /**
     * Get entity relation values.
     */
    public function fetchRelations(object $entity): array;

    /**
     * Map entity key->value to database specific column->value.
     * Original array also will be filtered: unused fields will be removed
     */
    public function mapColumns(array &$values): array;

    /**
     * Initiate chain of commands require to store object and it's data into persistent storage.
     *
     * @throws MapperException
     */
    public function queueCreate(object $entity, Node $node, State $state): CommandInterface;

    /**
     * Initiate chain of commands required to update object in the persistent storage.
     *
     * @throws MapperException
     */
    public function queueUpdate(object $entity, Node $node, State $state): CommandInterface;

    /**
     * Initiate sequence of of commands required to delete object from the persistent storage.
     *
     * @throws MapperException
     */
    public function queueDelete(object $entity, Node $node, State $state): CommandInterface;
}
```

The ORM will create a mapper using `Spiral\Core\FactoryInterface` which means your Mapper is able to request
dependencies available in the container associated with ORM Factory.

Some parameters will be provided by ORM itself, such as:

* **role** - entity role
* **schema** - entity schema
* **orm** - orm instance

You are able to use a single Mapper implementation for multiple entities.

## Database Mapper

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

class CustomMapper extends DatabaseMapper
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
            array_flip($this->columns)
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
