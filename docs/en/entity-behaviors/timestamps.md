# CreatedAt/UpdatedAt Timestamps

Probably the most popular of all behaviors, adds `created_at` and `updated_at` timestamp columns to your entity,
automatically saving datetime when a record is created or updated, respectively.

### Usage:

```php
use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Behavior;
use Cycle\ORM\Entity\Behavior;

#[Entity]
#[Behavior\CreatedAt(
    field: 'createdAt',   // Required. By default 'createdAt'
    column: 'created_at'  // Optional. By default 'null'. If not set, will be used information from property declaration.
)]
#[Behavior\UpdatedAt(
    field: 'updatedAt',   // Required. By default 'updatedAt' 
    column: 'updated_at'  // Optional. By default 'null'. If not set, will be used information from property declaration.
)]
class Page
{
    #[Column(type: 'primary')]
    private int $id;
    
    #[Column(type: 'datetime')]
    private \DateTimeImmutable $createdAt;
    
    #[Column(type: 'datetime', nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;
}
```

`Behavior\CreatedAt` and `Behavior\UpdatedAt` behaviors are able to declare columns `created_at` and `updated_at` of the 
required type and define them to the entity schema. So it's unnecessary to use `#[Column]` attribute for these fields. 

> Note: If you have a custom `created_at` or `updated_at` column declaration, it should be compatible 
> with `Behavior\CreatedAt` and `Behavior\UpdatedAt` column types, otherwise an exception will be thrown.
