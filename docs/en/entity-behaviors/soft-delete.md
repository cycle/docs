# DeletedAt (SoftDelete)

Adds a `deleted_at` column which defines if a record has been marked as deleted (and if so, when). Useful when designing
a highly complicated system where data consistency is important and even if some data should be invisible in the
backend, it should still remain in the database.

## Usage

```php
<?php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior\SoftDelete;

#[Entity]
#[Behavior\SoftDelete(
    field: 'deletedAt',   // Required. By default 'deletedAt' 
    column: 'deleted_at'  // Optional. By default 'null'. If not set, will be used information from property declaration.
)]
class Page
{
    #[Column(type: 'primary')]
    public int $id;
        
    #[Column(type: 'datetime', nullable: true)]
    public ?\DateTimeImmutable $deletedAt = null;
}
```

> Note: If you have a custom `deleted_at` column declaration, it should be compatible with `Behavior\SoftDelete` column 
> type, otherwise an exception will be thrown.
