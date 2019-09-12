# Using Bootstrap Toolkit
You can install Cycle using bootstrap toolkit for quick integrations. This tutorial assumes that your entity codebase is located in 
`src/` directory and accessible by Composer autoloader.

> Bundle comes with annotation support.

## Install
To install Console Toolkit:

```
$ composer require cycle/bootstrap
```

## Configure
In order to enable Console Toolkit we have to define bootstrap file which will configure your environment:

```php
<?php
// bootstrap.php

use Cycle\Bootstrap;
use Doctrine\Common\Annotations\AnnotationRegistry;

require_once "vendor/autoload.php";

AnnotationRegistry::registerLoader('class_exists');

// single database
$config = Bootstrap\Config::forDatabase(
    'sqlite:database.db', // connection dsn
    '',                   // username
    ''                    // password
);

// $config = Bootstrap\Config::forDatabase(
//    'mysql:host=127.0.0.1;dbname=database', // connection dsn
//    'mysql',                                // username
//    'mysql'                                 // password
// );

// which directory contains our entities
$config = $config->withEntityDirectory(__DIR__ . DIRECTORY_SEPARATOR . 'src');

// log all SQL messages to STDERR
// $config = $config->withLogger(new Bootstrap\StderrLogger(true));

// enable schema cache (use /vendor/bin/cycle schema:update to flush cache), keep commented to disable caching
// $config = $config->withCacheDirectory(__DIR__ . DIRECTORY_SEPARATOR . 'cache');

$orm = Bootstrap\Bootstrap::fromConfig($config);
```

## Console Commands
To enable console commands place file in `config/cycle-cli.php`:

```php
<?php
// config/cycle-cli.php
require_once 'bootstrap.php';
return $orm;
```

## Available Console commands
To display list of found entities:

```
$ /vendor/bin/cycle entity:list
```

To alter database schema to match entity declaration:

```
$ /vendor/bin/cycle schema:sync
```

To update Cycle without altering database schema (when cache is enabled):

```
$ /vendor/bin/cycle schema:update
```

To display list of available tables:
```
$ /vendor/bin/cycle db:list
```

To display the schema of a specific table:

```
$ /vendor/bin/cycle db:table {table-name}
```

> You can execute commands with `-vvv` flag to display SQL queries if logger is set.

# Example
Install the bundle and create `config/cycle-cli.php` and `bootstrap.php` files. Make sure that `composer.json` includes:

```json
"autoload": {
    "psr-4": {
      "": "src/"
    }
}
```

You can create your first entity in `src/`:

```php
<?php

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;

/**
 * @Entity()
 */
class User
{
    /** @Column(type="primary") */
    public $id;

    /** @Column(type="string") */
    public $name;
}
```

Generate database schema:

```bash
$ ./vendor/bin/cycle schema:sync
```

To use your entity create file `test.php`:

```php
<?php

use Cycle\ORM;

/** @var ORM\ORMInterface $orm */
include 'bootstrap.php';

$u = new \User();
$u->name = "Antony";

(new ORM\Transaction($orm))->persist($u)->run();

foreach ($orm->getRepository(User::class)->findAll() as $u) {
    print_r($u);
}
```

You can test the ORM now:

```bash
$ php test.php
```
