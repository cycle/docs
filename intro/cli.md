# Using Bootstrap Toolkit
You can install Cycle using a bootstrap toolkit for quick integrations. This tutorial assumes that your entity codebase is located in the
`src/` directory and accessible by the Composer autoloader.

> The bundle comes with annotation and proxies support.

## Install
To install Console Toolkit:

```
$ composer require cycle/bootstrap
```

## Configure
In order to enable Console Toolkit we have to define a bootstrap file which will configure your environment:

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
Files tree will look like this:
```ROOD_DIRECTORY/
├── src/
│   └── //app files
└── bootstrap.php
```
## Console Commands
To enable console commands, place the file in `config/cycle-cli.php`:

```php
<?php
// config/cycle-cli.php
require_once 'bootstrap.php';
return $orm;
```
Files tree will look like this:
```
ROOD_DIRECTORY/
├── src/
│   └── //app files
├── config/
│   └── config-cli.php
└── bootstrap.php
```
To display list of found entities:

```
$ ./vendor/bin/cycle entity:list
```

To alter the database schema to match entity declaration:

```
$ ./vendor/bin/cycle schema:sync
```

To update Cycle without altering the database schema (when cache is enabled):

```
$ ./vendor/bin/cycle schema:update
```

To display a list of available tables:
```
$ ./vendor/bin/cycle db:list
```

To display the schema of a specific table:

```
$ ./vendor/bin/cycle db:table {table-name}
```

> You can execute commands with the `-vvv` flag to display SQL queries if the logger is set.

# Example
Install the bundle and create `config/cycle-cli.php` and `bootstrap.php` files. Make sure that `composer.json` includes:

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
}
```

You can create your first entity in `src/`:

```php
<?php

namespace App;

use Cycle\Annotated\Annotation as Cycle;

/** @Cycle\Entity() */
class User
{
    /** @Cycle\Column(type="primary") */
    public $id;

    /** @Cycle\Column(type="string") */
    public $name;
}
```

Generate database schema:

```bash
$ ./vendor/bin/cycle schema:sync
```

To use your entity create the file `test.php`:

```php
<?php

use Cycle\ORM;

/** @var ORM\ORMInterface $orm */
include 'bootstrap.php';

$u = new \App\User();
$u->name = "Antony";

(new ORM\Transaction($orm))->persist($u)->run();

foreach ($orm->getRepository(\App\User::class)->findAll() as $u) {
    print_r($u);
}
```

You can test the ORM now:

```bash
$ php test.php
```
