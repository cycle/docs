# Mappers
Every entity in Cycle ORM must have associated Mapper object. Mappers responsible for entity creation, hydration and they must issue set of persist commands which later will be processed within the transaction.

## Interface
You can implement your own Mapper using `Cycle/ORM/MapperInterface`:

```php
use Cycle\ORM\Command\CommandInterface;
use Cycle\ORM\Command\ContextCarrierInterface;
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
     *
     * @return string
     */
    public function getRole(): string;
    /**
     * Init empty entity object an return pre-filtered data (hydration will happen on a later stage). Must
     * return tuple [entity, entityData].
     *
     * @param array $data
     * @return array
     */
    public function init(array $data): array;
    /**
     * Hydrate entity with dataset.
     *
     * @param object $entity
     * @param array  $data
     * @return object
     *
     * @throws MapperException
     */
    public function hydrate($entity, array $data);
    /**
     * Extract all values from the entity.
     *
     * @param object $entity
     * @return array
     */
    public function extract($entity): array;
    /**
     * Initiate chain of commands require to store object and it's data into persistent storage.
     *
     * @param object $entity
     * @param Node   $node
     * @param State  $state
     * @return ContextCarrierInterface
     *
     * @throws MapperException
     */
    public function queueCreate($entity, Node $node, State $state): ContextCarrierInterface;
    /**
     * Initiate chain of commands required to update object in the persistent storage.
     *
     * @param object $entity
     * @param Node   $node
     * @param State  $state
     * @return ContextCarrierInterface
     *
     * @throws MapperException
     */
    public function queueUpdate($entity, Node $node, State $state): ContextCarrierInterface;
    /**
     * Initiate sequence of of commands required to delete object from the persistent storage.
     *
     * @param object $entity
     * @param Node   $node
     * @param State  $state
     * @return CommandInterface
     *
     * @throws MapperException
     */
    public function queueDelete($entity, Node $node, State $state): CommandInterface;
}
```

ORM will create mapper using `Spiral\Core\FactoryInterface` which means you Mapper is able to request depenendencies available in
container associated with ORM Factory. 

Some parameters will be provided by ORM itself, such as: 
  * **role** - entity role
  * **schema** - entity schema
  * **orm** - orm instance
  
You are able to use single Mapper implement for multiple entities.

## Database Mapper
You can implement your own mapper to implement custom entity carrying model but still rely on common SQL functionality.
Let's define our model first:

```php
class Entity 
{
    private $data = [];
    
    public function __construct(array $data = [])
    {
        $this->data = $data;
    }
    
    public function setData(array $data)
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
class CustomMapper extends DatabaseMapper
{
    /** @var string */
    private $class;

   /**
     * @param ORMInterface $orm
     * @param string       $role
     */
    public function __construct(ORMInterface $orm, string $role)
    {
        parent::__construct($orm, $role);
        
        // entity class
        $this->class = $orm->getSchema()->define($role, Schema::ENTITY);
    }
    
    /**
     * @inheritdoc
     */
    public function init(array $data): array
    {
        $class = $this->class;
        return [new $class, $data];
    }
    
    /**
     * @inheritdoc
     */
    public function hydrate($entity, array $data)
    {
        $entity->setData($data);
        return $entity;
    }
    
    /**
     * @inheritdoc
     */
    public function extract($entity): array
    {
        return $entity->getData();
    }
    
    /**
     * Get entity columns.
     *
     * @param object $entity
     * @return array
     */
    protected function fetchFields($entity): array
    {
        // fetch entity fields and ignore custom columns
        return array_intersect_key(
            $this->extract($entity),
            array_flip($this->columns)
        );
    }
}
```

You can now create your entity and associate it with given mapper:

```php
/**
 * @entity(
 *    columns = {
 *        id: @column(type=primary)
 *    },
 *    mapper = "CustomMapper"
 * )
 */
class User extends Entity
{

}
```

Update your ORM schema to register entity. You can use your entity freely after this operation.

> ORM can work with different types or entites within one system.

## ActiveRecord 
Similar approach can be used to implement AR like entities. You would have to expose global instance of ORM in order to gain access to it
from your entity `save()` and `delete()` methods:

```php
class Entity 
{
    // ...
    
    public function save()
    {
        $orm = App::getORM();
        (new Transaction($orm))->persist($this)->run();
    }
   
    public function delete()
    {
        $orm = App::getORM();
        (new Transaction($orm))->delete($this)->run();
    }
}
```
