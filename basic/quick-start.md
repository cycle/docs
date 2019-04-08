# Quick Start
This guide provides quick overview of ORM installation and configuration process and example using annotateed entity. other sections of the documentation will provide deeper insign of various use-cases.

## Installation
Cycle ORM requiments include:
  * PHP7.1+
  * PHP-PDO
  * PDO drivers for desired databases

Cycle ORM is available as composer repository and can be installed using following command in a root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request additional package:

```bash
$ composer require cycle/annotated
```

Given commands will download Cycle ORM dependencies such as `spiral/databses`, `doctrine/collections` and `zendframework/zend-hydrator`.

In order to access to Cycle ORM classes make sure to include `vendor/autoload.php` into your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Configuration
In order to operate, Cycle ORM require proper database connection to be set. All database connections are managed using `DatabaseManager` service provided by the package `spiral/database`. We can configure our first database connection to be initiated on demand using following configuration:

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';

$dbal = new Database\DatabaseManager(new Database\Config\DatabaseConfig([
    'default'     => 'default',
    'databases'   => [
        'default' => [
            'connection' => 'sqlite'
        ]
    ],
    'connections' => [
        'sqlite' => [
            'driver'  => Database\Driver\SQLite\SQLiteDriver::class,
            'options' => [
                'connection' => 'sqlite:database.db',
                'username'   => '',
                'password'   => '',
            ]
        ]
    ]
]));
```

> Read about how to connect to other database types in [next section](connection.md). You can also configure
database connections in runtime.

We can get access to our database object and check connection:

```php
print_r($dbal->database('default')->getTables());
```

> Run `php filename.php`, the result must be empty array.

## ORM
ORM service can be initiated using following construction:

```php
$orm = new ORM\ORM(new ORM\Factory(
    $dbal,
    ORM\Config\RelationConfig::getDefault()
));
```

> Make sure to add `use Cycle\ORM;` at top of your file.

ORM is ready for use but it does not have any entity associated with it.

## Register Namespace
We can create our first entity in a directory `src` of our project.

Register new namespace in your composer.json file:

```json
"autoload": {
    "psr-4": {
      "Example\\": "src/"
    }
  }
```

Execute: 

```bash
$ composer dump
```

## Create Entity
We can now create our first entity in `src` folder. We will use capabilities provided by `cycle/annoted` package to describe our schema:

```php
<?php declare(strict_types=1);

namespace Example;

/**
 * @entity
 */
class User
{
    /**
     * @column(type=primary)
     * @var int
     */
    protected $id;

    /**
     * @column(type=string)
     * @var string
     */
    protected $name;
    
    public function getId(): int
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function setName(string $name): void
    {
        $this->name = $name;
    }
}
```

Cycle will automatically assign role `user` and table `users` from default database to this entity.

## Schema Generation
In order to operate we need to generate ORM Schema which will describe how our entities are configured. Thought we can do it manually 
we will use pipeline generator provided by `cycle/schema-builder` package and generators from `cycle/annotated`.

First, we have to create instance of `ClassLocator` which will automatically find needed entities:

```php
$classLocator = (new \Spiral\Tokenizer\Tokenizer(new \Spiral\Tokenizer\Config\TokenizerConfig([
    'directories' => ['src/']
])))->classLocator();
```

We can immediatelly check if our class visible for static indexation:

```php
print_r($classLocator->getClasses());
```

Once class locator is established we can create our schema generation pipeline. First, we will add needed namespace imports:

```php
use Cycle\Schema;
use Cycle\Annotated;
```

Now we can define our pipeline:

```php
$schema = (new \Cycle\Schema\Compiler())->compile(new \Cycle\Schema\Registry($dbal), [
    new Annotated\Entities($cl),
    new Schema\Generator\ResetTables(),
    new Schema\Generator\GenerateRelations(),
    new Schema\Generator\ValidateEntities(),
    new Schema\Generator\RenderTables(),
    new Schema\Generator\RenderRelations(),
    new Schema\Generator\SyncTables(),
    new Schema\Generator\GenerateTypecast(),
]);
```

> We will explain what each iterator is doing in a later sections. Please note, while complining your schema `SyncTables` will automatically adjust your database structure! Do not use it on real database!

The resulted schema can be passed to ORM. 

```php
$orm = $orm->withSchema(new ORM\Schema($schema));
```

Your ORM is ready to be used. 

> You can dump `schema` variable to check the internal representation of your entity schema.

## Create First Entity
Now, we can create and save our first entity in database:


## Create your entity
We can create your 
> Generated schema is intended to be cached in your application, only re-generate schema when it's needed.

```php
$u = new \Example\User();
$u->setName("Hello World");
```

To persist our entity we have pass it into Transaction object:

```php
$t = new ORM\Transaction($orm);
$t->persist($u);
$t->run();
```

You can immediatelly dump your entity to see newly generated primary key:

```php
print_r($u);
```

## Select Entity
You can select the entity from database using it's primary key and associated repository:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);
```

> Remove code from section above to avoid fetching same entity as one which was created above.

## Update Entity

To update entity data simple change it's value before persisting it in the transaction:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);

print_r($u);
$u->setName("New " . mt_rand(0, 1000));

(new ORM\Transaction($orm))->persist($u)->run();
```

You can notice new name being displayed on every script iteration.

## Delete Entity
To delete entity simply call method `delete` of the Transation:

```php
(new ORM\Transaction($orm))->delete($u)->run();
```

## Update Entity Schema
You can modify your entity schema to add new columns. Note that you have to either specify default value or set column as `nullable` in order to apply modification to non empty table.

```php
/**
* @column(type=int,nullable=true)
* @var int|null
*/
protected $age;
```

Schema will be automatically updated on next script invocation.
