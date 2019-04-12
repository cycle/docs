# Create, Update and Delete entities
Any persitence operation with entity or entities have to be done using `Cycle\ORM\Transaction` object.

## Create Entity
In order to create an entity simply pass it's instance to the transaction object and invoke method `run`:

```php
$user = new User();
$user->setName("Antony");

$tr = new Transaction($orm);
$tr->persist($user);
$tr->run();
```

In order to process persistent errors make sure to handle exceptions produced by `run` method:

```php
try {
   $tr->run();
} catch (\Throwable $e) {
   print_r($e);
}
```

One of the most important type of exception you must handle is `Spiral\Database\Exception\DatabaseException`. This exception branched
into multiple types for each of the error types:

```php
try {
   $tr->run();
} catch (ConnectionException $e) {
   print_r("database has gone away");
} catch (ConstrainException $e) {
   print_r("database constrain not met, nulable field?");
}
```

## Update the Entity
In order to update the entity you must first obtain the loaded entity object:

```php
$user = $orm->getRepository(User::class)->findByPK(1);
```

Simply change desired entity fields and register it in the transaction using `persist` method:

```php
$user->setName("John");

$tr = new Transaction($orm);
$tr->persist($user);
$tr->run();
```

Note, by default, ORM will update only changed entity fields (a.k.a. dirty state), given code would produce
SQL code similar to:

```sql
UPDATE `users` SET `name` = "John" WHERE `id` = 1 
```

## Delete Entity
Any entity can be deleted using transaction method `delete`:

```php
$user = $orm->getRepository(User::class)->findByPK(1);

$tr = new Transaction($orm);
$tr->delete($user);
$tr->run();
```

Please note, ORM would not automatically trigger the delete operation for related entities and will rely 
on foreign key rules set in database.

## Persisting related Entities
Persinting entity will also persist all related entities within it.
 
```php
$user = new User();
$user->setAddress(new Address());
$user->getAddress()->setCountry("USA");

$tr = new Transaction($orm);
$tr->persist($user);
$tr->run();

print_r($user->getAddress()->getID());
```

This behaviour is enabled by default by persisting entity with `Transaction::MODE_CASCADE` flag.
Code above can be equally rewtitted as:

```php
$tr = new Transaction($orm);
$tr->persist($user, Transaction::MODE_CASCADE);
$tr->run();
```

Pass `Transaction::MODE_ENTITY_ONLY` flag to disable casacade persising of related entities:

```php
$tr = new Transaction($orm);
$tr->persist($user, Transaction::MODE_ENTITY_ONLY);
$tr->run();
```

> `Transaction::MODE_ENTITY_ONLY` flag can be used while creating or updating the entity.

You can also turn off cascading on relation level by setting `cascade` flag to `false`.
