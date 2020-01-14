# Quick Start
This guide provides a quick overview of the ORM's installation and configuration process, and an example using an annotated entity. Other sections of the documentation will provide deeper insight into various use-cases.

> Use [Bootstrap](/intro/cli.md) to automatically configure Cycle and get access to quick CLI commands.

## Requirements
  * PHP 7.2+
  * PHP-PDO
  * PDO drivers for desired databases

## Installation
Cycle ORM is available as a composer package and can be installed using the following command in the root of your project:

```bash
$ composer require cycle/orm
```

In order to enable support for annotated entities you have to request an additional package:

```bash
$ composer require cycle/annotated
```

> You can skip this step and define the mapping schema manually, see above.

This command will also download Cycle ORM dependencies such as `spiral/database`, `doctrine/collections` and `zendframework/zend-hydrator`.

In order to access Cycle ORM classes, make sure to include `vendor/autoload.php` in your file.

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';
```

## Configuration
In order to operate, Cycle ORM requires a proper database connection to be set. All database connections are managed using the `DatabaseManager` service provided by the package `spiral/database`. We can configure our first database connection to be initiated on demand using the following configuration:

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

> Read about how to connect to other database types in [this section](/basic/connect.md). You can also configure
database connections at runtime.

Check database access using following code:

```php
print_r($dbal->database('default')->getTables());
```

> Run `php {filename}.php`, the result must be empty array.

### ORM
Initiate ORM service:

```php
$orm = new ORM\ORM(new ORM\Factory($dbal));
```

> Make sure to add `use Cycle\ORM;` at top of your file.

### Register Namespace
We can create our first entity in a directory `src` of our project.

Register the new namespace in your composer.json file:

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

### Manually Configure Mapping
You can avoid using `cycle/annotated` and ignore sections "Define Entity" and "Schema Generation". To do that we can define the ORM schema manually, right in PHP code:

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

You can use the ORM now:

```php
$user = new User();
$user->setName("John");

(new Transaction($orm))->persist($user)->run();
```

> Note, in this case, ORM can not automatically migrate your database schema.

Read more about other ways to declare a mapping schema in later sections of the ORM documentation (for example [dynamic mapping](https://github.com/cycle/docs/blob/master/advanced/dynamic-schema.md#example)).

## Define Entity
To create our first entity (in the `src` folder) we will use the capabilities provided by the `cycle/annotated` package to describe our desired schema:

```php
<?php declare(strict_types=1);

namespace Example;

use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

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

Cycle will automatically assign the role `user` and table `users` from the default database to this entity.

> Attention, `@Entity` annotation is required!

### Schema Generation
In order to operate we need to generate an ORM Schema which will describe how our entities are configured. Though we can do it manually, we will use the pipeline generator provided by `cycle/schema-builder` package, and generators from `cycle/annotated`.

First, we have to create instance of `ClassLocator` which will automatically find the required entities:

```php
$finder = (new \Symfony\Component\Finder\Finder())->files()->in([__DIR__]);
$classLocator = new \Spiral\Tokenizer\ClassLocator($finder);
```

We can immediately check if our class is visible (ClassLocator will perform static indexation of your code behind the hood):

```php
print_r($classLocator->getClasses());
```

Once the class locator is established we can create our schema generation pipeline. First, we will add the required namespace imports:

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
    new Schema\Generator\ResetTables(),       // re-declared table schemas (remove columns)
    new Annotated\Embeddings($classLocator),  // register embeddable entities
    new Annotated\Entities($classLocator),    // register annotated entities
    new Annotated\MergeColumns(),             // add @Table column declarations
    new Schema\Generator\GenerateRelations(), // generate entity relations
    new Schema\Generator\ValidateEntities(),  // make sure all entity schemas are correct
    new Schema\Generator\RenderTables(),      // declare table schemas
    new Schema\Generator\RenderRelations(),   // declare relation keys and indexes
    new Annotated\MergeIndexes(),             // add @Table column declarations
    new Schema\Generator\SyncTables(),        // sync table changes to database
    new Schema\Generator\GenerateTypecast(),  // typecast non string columns
]);
```

> We will explain what each generator is doing in later sections. Please note, while computing your schema `SyncTables` will automatically adjust your database structure! Do not use it on a real database!

The resulted schema can be passed to the ORM.

```php
$orm = $orm->withSchema(new ORM\Schema($schema));
```

> The generated schema is intended to be cached in your application, only re-generate schema when it's needed.

Your ORM is now ready for use.

> You can dump the `schema` variable to check the internal representation of your entity schema.

## Work with Entity
You can start working with entities once your ORM is fully configured.

### Persist Entity
Now we can init and persist our first entity in the database:

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

You can immediately dump the object to see newly generated primary key:

```php
print_r($u);
```

### Select Entity
You can select the entity from the database using its primary key and associated repository:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);
```

> Remove the code from the section above to avoid fetching the entity from memory.

### Update Entity
To update the entity data simply change its value before persisting it in the transaction:

```php
$u = $orm->getRepository(\Example\User::class)->findByPK(1);
print_r($u);

$u->setName("New " . mt_rand(0, 1000));

(new ORM\Transaction($orm))->persist($u)->run();
```

Notice how a new name will be displayed on every script iteration.

### Delete Entity
To delete the entity simply call the method `delete` of the Transaction:

```php
(new ORM\Transaction($orm))->delete($u)->run();
```

## Update Entity Schema
You can modify your entity schema to add new columns. Note that you have to either specify a default value or set the column as `nullable` in order to apply the modification to the non empty table.

```php
/**
* @Column(type=int,nullable=true)
* @var int|null
*/
protected $age;
```

The schema will be automatically updated on the next script invocation. We can find all users with undefined age using the following method:

```php
$users = $orm->getRepository(\Example\User::class)->findAll(['age' => null]);
print_r($users);
```
