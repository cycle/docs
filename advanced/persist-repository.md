# Persisting Repositories
By default ORM design, the Repository object is used only for Select logic (read-only). Write operations are controlled via Transactions
(entity -> transaction -> mapper -> command -> storage).

However, it is possible to safely add a `save` or `delete` method to your repositories to avoid usage of transactions in the application code.

## Use Repositories with Transaction
We can create a simple `save` method in the Repository, which will save the entity's current state and it's loaded relations or entity only.
In order to do that we have to create a transaction inside our object:

```php
class UserPersistRepository extends Repository
{
    /** @var Transaction */
    private $transaction;

    /**
     * @param Select       $select
     * @param ORMInterface $orm
     */
    public function __construct(Select $select, ORMInterface $orm)
    {
        parent::__construct($select);
        $this->transaction = new Transaction($orm);
    }

    /**
     * @param User $user
     * @param bool $cascade
     *
     * @throws \Throwable
     */
    public function save(User $user, bool $cascade = true)
    {
        $this->transaction->persist(
            $user,
            $cascade ? Transaction::MODE_CASCADE : Transaction::MODE_ENTITY_ONLY
        );

        $this->transaction->run(); // transaction is clean after run
    }
}
```

You can associate the repository to your entity via the annotation `@Entity(repository="UserPersistRepository")`, or manually:

```php
$orm = $orm->withSchema(new Schema([
    User::class => [
        Schema::ROLE        => 'user',
        Schema::MAPPER      => Mapper::class,
        Schema::REPOSITORY  => UserPersistRepository::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => ['id', 'email'],
        Schema::RELATIONS   => []
    ]
]));
```

You can use the repository now to create/update entity in your application:

```php
/** @var UserPersistRepository $users */
$users = $orm->getRepository(User::class);

$u = new User();
$u->email = "test@email.com";

$users->save($u);
```
