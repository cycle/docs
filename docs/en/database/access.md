# Database - Access Database

Follow the configuration instructions [here](/docs/en/database/configuration.md).

## Access the Database

Once DBAL component configured properly, you can access your databases in controllers and services multiple ways:

```php
declare(strict_types=1);

require_once "vendor/autoload.php";

use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\DatabaseManager;
use Cycle\Database\Driver\SQLite\SQLiteDriver;

$dbal = new DatabaseManager(new DatabaseConfig(...));

//Default database
var_dump($dbal->database());

//Using alias default which points to primary database
var_dump($dbal->database('default'));

//Secondary
var_dump($dbal->database('slave'));
```

## Run Queries

To run the database query use method `query`:

```php
$db = $dbal->database();

var_dump(
    $db->query(
        'SELECT * FROM users WHERE id > ?',
        [
            1
        ]
    )->fetchAll()
);
```

To execute update or delete statement use the alternative method `execute`:

```php
$db = $dbal->database();

var_dump(
    $db->execute(
        'DELETE FROM users WHERE id > ?',
        [
            1
        ]
    ) // number of affected rows 
);
```

> Read how to use query builders [here](/docs/en/database/query-builders.md).
