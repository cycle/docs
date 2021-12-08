# Persisting Repositories

By default ORM design, the Repository object is used only for Select logic (read-only). Write operations are controlled
via EntityManager (entity -> entity manager -> mapper -> command -> storage).

However, it is possible to safely add a `save` or `delete` method to your repositories to avoid usage of transactions in
the application code.

## Use Repositories with entity manager

We can create a simple `save` method in the Repository, which will save the entity's current state and it's loaded
relations or entity only. In order to do that we have to create an Entity manager inside our object:

```php
use Cycle\ORM\Select;
use Cycle\ORM\EntityManager;
use Cycle\ORM\ORMInterface;

class UserPersistRepository extends Select\Repository
{
    private EntityManager $entityManager;

    public function __construct(Select $select, ORMInterface $orm)
    {
        parent::__construct($select);
        $this->entityManager = new EntityManager($orm);
    }

    public function save(User $user, bool $cascade = true)
    {
        $this->entityManager->persist(
            $user,
            $cascade
        );

        // entity manager is clean after run
        $this->entityManager->run();
    }
}
```

> Read more about entity manager [here](/docs/en/advanced/entity-manager.md).

You can associate the repository to your entity via the attribute `#[Entity(repository: UserPersistRepository::class)]`,
or manually:

```php
use Cycle\ORM\Schema;
use Cycle\ORM\Mapper\Mapper;

$orm = $orm->with(schema: new Schema([
    User::class => [
        Schema::ROLE => 'user',
        Schema::MAPPER => Mapper::class,
        Schema::REPOSITORY => UserPersistRepository::class,
        Schema::DATABASE => 'default',
        Schema::TABLE => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS => ['id', 'email'],
        Schema::RELATIONS => [],
    ]
]));
```

You can use the repository now to create/update entity in your application:

```php
/** @var UserPersistRepository $users */
$users = $orm->getRepository(User::class);

$user = new User();
$user->email = "test@email.com";

$users->save($user);
```
