# Using Console Toolkit
You can install Cycle using Console toolkit for quick integrations.

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
    'sqlite:database.db',
    '',
    ''
);

// which directory contains our entities
$config = $config->withEntityDirectory(__DIR__ . DIRECTORY_SEPARATOR . '..' . DIRECTORY_SEPARATOR . 'src');

// log all SQL messages to STDERR
$config = $config->withLogger(new Bootstrap\StderrLogger(true));

// enable schema cache (use /vendor/bin/cycle schema:update to flush cache)
//$config = $config->withCacheDirectory(__DIR__ . DIRECTORY_SEPARATOR . '..' . DIRECTORY_SEPARATOR . 'cache');

$orm = Bootstrap\Bootstrap::fromConfig($config);
```

To enable console commands place file in `config/cycle-cli.php`:

```php
<?php
// config/cycle-cli.php
require_once 'bootstrap.php';
return $orm;
```
