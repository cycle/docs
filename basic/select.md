# Select Entity
Cycle ORM provide multiple options to select entity data from the database. 
The most common and recommended methods to use associated entity repository.

## Using Repository
To access repository associated with specific entity use method `getRepository` of orm service:

```php
$r = $orm->getRepository(User::class);
```

You can request repository instance using entity class name or it's role name:

```php
$r = $orm->getRepository("user");
```

The Repository provides multiple methods to select the entity.

