# Using Console Toolkit
You can install Cycle using Console toolkit for quick integrations. This tutorial assumes that your entity codebase is located in 
`src/` directory and accessible by Composer Autoloader.

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
declare(strict_types=1);

use Cycle\Bootstrap;

require_once "vendor/autoload.php";

// single database
$config = Bootstrap\Config::forDatabase(
    'sqlite:database.db', // connection dsn
    '',                   // username
    ''                    // password
);

// which directory contains our entities
$config = $config->withEntityDirectory(__DIR__ . DIRECTORY_SEPARATOR . 'src');

// log all SQL messages to STDERR
$config = $config->withLogger(new Bootstrap\StderrLogger(true));

// enable schema cache (use /vendor/bin/cycle schema:update to flush cache), keep commented to disable caching
//$config = $config->withCacheDirectory(__DIR__ . DIRECTORY_SEPARATOR . 'cache');

$orm = Bootstrap\Bootstrap::fromConfig($config);
```

To enable console commands place file in `config/cycle-cli.php`:

```php
<?php
// config/cycle-cli.php
require_once 'bootstrap.php';
return $orm;
```

You can create your entities now.

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
