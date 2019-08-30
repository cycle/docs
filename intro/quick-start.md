# Quick Start
This guide provides a quick overview of ORM installation, configuration process and an example using an annotated entity. Other sections of the documentation will provide deeper insign of various use-cases.

> Use [Bootstrap](/intro/cli.md) to automatically configure Cycle and get access to quick CLI commands.

## Requirements
  * PHP 7.2+
  * PHP-PDO
  * PDO drivers for desired databases
  
## Installation
Cycle ORM is available as composer repository and can be installed using the following command in the root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request an additional package:

```bash
$ composer require cycle/annotated
```

> You can skip this step and define mapping schema manually, see above.

This command will also download Cycle ORM dependencies such as `spiral/database`, `doctrine/collections` and `zendframework/zend-hydrator`.

In order to access Cycle ORM classes make sure to include `vendor/autoload.php` in your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Configuration
In order to operate, Cycle ORM require proper database connection to be set. All database connections are managed using `DatabaseManager` service provided by the package `spiral/database`. We can configure our first database connection to be initiated on demand using the following configuration:

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';

use Spiral\Database;

$dbal = new Database\DatabaseManager(
    new Database\Config\DatabaseConfig([
        'default'     => 'default',
        'databases'   => [
            'default' => ['connection' => 'sqlite']
        ],
        'connections' => [
            'sqlite' => [
                'driver'  => Database\Driver\SQLite\SQLiteDriver::class,
                'connection' => 'sqlite:database.db',
                'username'   => '',
                'password'   => '',
            ]
        ]
    ])
);
```

> Read about how to connect to other database types in [next section](/basic/connect.md). You can also configure
database connections at runtime.

Check database access using following code:

```php
print_r($dbal->database('default')->getTables());
```

> Run `php {filename}.php`, the result must be empty array.

## ORM
Initiate ORM service:

```php
$orm = new ORM\ORM(new ORM\Factory($dbal));
```

> Make sure to add `use Cycle\ORM;` at top of your file.

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

## Manually Configure Mapping
You can avoid using `cycle/annotated` and ignore sections "Define Entity", "Schema Generation". To do that we can define ORM schema manually, right in PHP code:

```php
$orm = $orm->withSchema(new Schema([
    'user' => [
         Schema::MAPPER      => Mapper::class, // default POPO mapper
         Schema::ENTITY      => User::class, 
         Schema::DATABASE    => 'default',
         Schema::TABLE       => 'users',
         Schema::PRIMARY_KEY => 'id',
         Schema::COLUMNS     => [
            'id'   => 'id',  // property => column 
            'name' => 'name'
         ],
         Schema::TYPECAST    => [
            'id' => 'int'
         ],
         Schema::RELATIONS   => []
     ]
]));
```

You can use ORM now:

```php
$user = new User();
$user->setName("John");

(new Transaction($orm))->persist($user)->run();
```

> Note, in this case ORM can not automatically migrate your database schema.

Read more about other ways to declare mapping schema in later sections of ORM documentation (for example [dynamic mapping](https://github.com/cycle/docs/blob/master/advanced/dynamic-schema.md#example)). 

## Define Entity
To create our first entity in `src` folder we will use capabilities provided by `cycle/annotated` package to describe desired schema:

```php
<?php declare(strict_types=1);

namespace Example;

/**
 * @Entity
 */
class User
{
    /**
     * @Column(type="primary")
     * @var int
     */
    protected $id;

    /**
     * @Column(type="string")
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

> Attention, `@Entity` annotation is required!

## Schema Generation
In order to operate we need to generate ORM Schema which will describe how our entities are configured. Thought we can do it manually 
we will use the pipeline generator provided by `cycle/schema-builder` package and generators from `cycle/annotated`.

First, we have to create instance of `ClassLocator` which will automatically find needed entities:

```php
$finder = (new \Symfony\Component\Finder\Finder())->files()->in([__DIR__]));
$classLocator = new \Spiral\Tokenizer\ClassLocator($finder);
```

We can immediatelly check if our class visible (ClassLocator will perform static indexation of your code behind the hood):

```php
print_r($classLocator->getClasses());
```

Once class locator is established we can create our schema generation pipeline. First, we will add needed namespace imports:

```php
use Cycle\Schema;
use Cycle\Annotated;
use Doctrine\Common\Annotations\AnnotationRegisty;
```

Now we can define our pipeline:

```php
// autoload annotations
AnnotationRegistry::registerLoader('class_exists');

$schema = (new Schema\Compiler())->compile(new Schema\Registry($dbal), [
    new Annotated\Embeddings($cl),            // register embeddable entities
    new Annotated\Entities($cl),              // register annotated entities
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

> We will explain what each generator is doing in later sections. Please note, while complining your schema `SyncTables` will automatically adjust your database structure! Do not use it on a real database!

The resulted schema can be passed to ORM. 

```php
$orm = $orm->withSchema(new ORM\Schema($schema));
```

> Generated schema is intended to be cached in your application, only re-generate schema when it's needed.

Your ORM is ready for use. 

> You can dump `schema` variable to check the internal representation of your entity schema.

## Persist Entity
Now, we can init and persist our first entity in database:

```php
$u = new \Example\User();
$u->setName("Hello World");
```

To persist our entity we have register it in the transaction:

```php
$t = new ORM\Transaction($orm);
$t->persist($u);
$t->run();
```

You can immediatelly dump the object to see newly generated primary key:

```php
print_r($u);
```

## Select Entity
You can select the entity from database using it's primary key and associated repository:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);
```

> Remove the code from the section above to avoid fetching entity from memory.

## Update Entity
To update entity data simply change it's value before persisting it in the transaction:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);
print_r($u);

$u->setName("New " . mt_rand(0, 1000));

(new ORM\Transaction($orm))->persist($u)->run();
```

You can notice that a new name will be displayed on every script iteration.

## Delete Entity
To delete entity simply call method `delete` of the Transation:

```php
(new ORM\Transaction($orm))->delete($u)->run();
```

## Update Entity Schema
You can modify your entity schema to add new columns. Note that you have to either specify a default value or set column as `nullable` in order to apply the modification to the non empty table.

```php
/**
* @Column(type=int,nullable=true)
* @var int|null
*/
protected $age;
```

Schema will be automatically updated on next script invocation. We can find all users with non defined age using following method:

```php
$users = $orm->getRepository(\Example\User::class)->findAll(['age' => null]);
print_r($users);
```
