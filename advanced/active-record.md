# Active Record
Cycle ORM leaves enough room for developers to alter its behavior. In this section, we will try to create ActiveRecord-like entity implementation which is automatically configured based on database introspection. 

> Engine is still going to use repositories and mappers behind the hood. In this article we are only going to handle column mapping, relation configuration must be done separately.

We will try to achive the following functionality:

```php
$u = User::find()->findOne(['name' => "John"]) ?? new User();
$u->name = "new name";
$u->save();
```

## Entity
We would try to simplify our entity by disabling the hydration, properties will be stored in the form of an array. We are also going to create `save`, `find` and `delete` methods.

First, we will have to ensure access to the ORM instance, a transaction, and the repository (statically):

```php
use Cycle\ORM;

abstract class Record
{
    private static $orm;

    public function save(bool $saveChildren = true)
    {
        $tr = new ORM\Transaction(self::getORM());
        $tr->persist(
            $this, 
            $saveChildren ? ORM\Transaction::MODE_CASCADE : ORM\Transaction::MODE_ENTITY_ONLY
        );
        $tr->run();
    }

    public function delete()
    {
        $tr = new ORM\Transaction(self::getORM());
        $tr->delete($this);
        $tr->run();
    }

    public static function find(): ORM\RepositoryInterface
    {
        return self::getORM()->getRepository(static::class);
    }

    public static function getORM(): ORM\ORMInterface
    {
        return self::$orm;
    }

    public static function setORM(ORM\ORMInterface $orm)
    {
        self::$orm = $orm;
    }
}
```

To store and access entity data we are going to use private array (prefix `__` added to avoid collisions with user methods):

```php
abstract class Record
{
    private $data = [];

    public function __construct(array $data = [])
    {
        $this->__setData($data);
    }

    public function __setData(array $data)
    {
        $this->data = $data;
    }

    public function __getData(): array
    {
        return $this->data;
    }

    // ...
}
```

We can use `__get` and `__set` to access our data, the resulted base entity will look like:

```php
abstract class Record
{
    private static $orm;

    private $data = [];

    public function __construct(array $data = [])
    {
        $this->__setData($data);
    }

    public function __setData(array $data)
    {
        $this->data = $data;
    }

    public function __getData(): array
    {
        return $this->data;
    }

    public function __get($name)
    {
        return $this->data[$name];
    }

    public function __set($name, $value)
    {
        $this->data[$name] = $value;
    }

    public function save(bool $saveChildren = true)
    {
        $tr = new ORM\Transaction(self::getORM());
        $tr->persist(
            $this, 
            $saveChildren ? ORM\Transaction::MODE_CASCADE : ORM\Transaction::MODE_ENTITY_ONLY
        );
        $tr->run();
    }

    public function delete()
    {
        $tr = new ORM\Transaction(self::getORM());
        $tr->delete($this);
        $tr->run();
    }

    public static function find(): ORM\RepositoryInterface
    {
        return self::getORM()->getRepository(static::class);
    }

    public static function getORM(): ORM\ORMInterface
    {
        return self::$orm;
    }

    public static function setORM(ORM\ORMInterface $orm)
    {
        self::$orm = $orm;
    }
}
```

## Mapper
Now, in order to properly initiate and persist entity, we have to create mapper specific to our implementation. We can extend `Cycle\ORM\Mapper\DatabaseMapper`
for these purposes:

```php
use Cycle\ORM;

class ARMapper extends ORM\Mapper\DatabaseMapper
{
    public function init(array $data): array
    {
        $class = $this->orm->getSchema()->define($this->role, ORM\Schema::ENTITY);

        // return empty entity and prepared data
        return [new $class, $data];
    }

    public function hydrate($entity, array $data)
    {
        /** @var Record $entity */
        $entity->__setData($data);
        
        return $entity;
    }

    public function extract($entity): array
    {
        /** @var Record $entity */
        return $entity->__getData();
    }

    protected function fetchFields($entity): array
    {
        // ignore properties which are not declated in schema
        return array_intersect_key(
            $this->extract($entity),
            array_flip($this->columns)
        );
    }
}
```

> You are free to use any other mapper customizations in combination with AR approach.

## Table and Entity
Since we are using the AR approach we are going to use table introspection to drive mapping schema (which can be cached). 

Assuming we have table `user` with columns (id, name) we can create our first entity `User`:

```php
class User extends Record {
    public const TABLE = 'user';
}
```

## Generation
Next step is to create generator capable of finding and configuring our entity. We are going to use `Spiral\Tokenizer\ClassesInterface` 
to automatically locate our `Record` classes:

```php
use Cycle\Schema\GeneratorInterface;
use Cycle\Schema\Registry;
use Spiral\Prototype\Traits\PrototypeTrait;
use Spiral\Tokenizer\ClassesInterface;

class ARGenerator implements GeneratorInterface
{
    /** @var ClassesInterface */
    private $classLocator;

    public function __construct(ClassesInterface $classLocator)
    {
        $this->classLocator = $classLocator;
    }

    public function run(Registry $registry): Registry
    {
        foreach ($this->classLocator->getClasses(Record::class) as $entity) {
            if ($entity->isAbstract()) {
                continue;
            }

            $this->declareEntity($registry, $entity->getName(), $entity->getConstant('TABLE'));
        }

        return $registry;
    }

    private function declareEntity(Registry $registry, string $class, string $table)
    {
        // see below
    }
}
```

In our `declareEntity` method we have to associate table with our entity and fetch all available columns:

```php
use Cycle\Schema\Definition;

// ...

private function declareEntity(Registry $registry, string $class, string $table)
{
    $e = new Definition\Entity();
    $e->setRole($class);
    $e->setClass($class);
    $e->setMapper(ARMapper::class);

    $registry->register($e);
    $registry->linkTable($e, 'default', $table);

    $schema = $registry->getTableSchema($e);

    foreach ($schema->getColumns() as $column) {
        $field = new Definition\Field();
        $field->setColumn($column->getName());

        if (in_array($column->getName(), $schema->getPrimaryKeys())) {
            $field->setPrimary(true);
        }

        $e->getFields()->set($column->getName(), $field);
    }
}
```

> Error checking is ommited.

## Generating Schema
Now we have all the pieces to compile our ORM mapping schema:

```php
$finder = (new \Symfony\Component\Finder\Finder())->files()->in(['src-directory']);
$classLocator = new \Spiral\Tokenizer\ClassLocator($finder);

$schema = (new Compiler())->compile(
    new Registry($orm->getFactory()),
    [
        new ARGenerator($classLocator),
        new ValidateEntities(),
        new GenerateTypecast()
    ]
);

$schema = new Schema($schema);
```

> You can store generated schema in the cache to speed up application bootstrap.

## Write Schema Manually
You can also write ORM schema manually for your AR entities, it will allow you to skip compilation and caching phases all together:

```php
use Cycle\ORM\Schema;

$schema = new Schema([
    'user' => [
        Schema::ENTITY      => User::class,
        Schema::MAPPER      => ARMapper::class,
        Schema::REPOSITORY  => UserRepository::class, // optional, avaible via User::find() 
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => [
            'id'   => 'id',  // property => column
            'name' => 'name'
        ],
        Schema::RELATIONS   => []
    ]
]);
```

## Using Active Records
Once the schema is obtained we can start working with our entities:

```php
$orm = $orm->withSchema($schema);
Record::setORM($orm);
```

To create user:

```php
$u = new User();
$u->name = 'Antony';
$u->save();

print_r($u->id);
```

To find users:

```php
foreach (User::find()->findAll() as $u) {
    print_r($u);
}
```
