# Profiling and Logging

Cycle Database does not provide logging abilities on the core level, however, it provides multiple points at which logger can be integrated.

Both `DatabaseManager` and each of the database drivers implements the `Psr\Log\LoggerAwareInterface` and you can enable SQL logging by assigning a logger:

#### Globally for each driver
```php
$dbal = new DatabaseManager(
    new DatabaseConfig(...)
);

$dbal->setLogger($myLogger);
```

#### For specific driver
```php
$driver = $dbal->database('default')->getDriver();
$driver->setLogger($myLogger);
```

#### Globally via Logger factory

```php
use Cycle\Database\DatabaseManager;
use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\LoggerFactoryInterface;

class CustomLoggerFactory implements LoggerFactoryInterface {

    public function getLogger(DriverInterface $driver = null): LoggerInterface
    {
        if ($driver->getType() === 'SQLite') {
            return new FileLooger();
        }
        
        return new \Psr\Log\NullLogger();
    }
}

$dbal = new DatabaseManager(
    new DatabaseConfig(...), 
    new CustomLoggerFactory()
);
```
