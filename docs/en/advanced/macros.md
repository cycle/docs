# Macros

Расширяя мапперы мы можем очень гибко автоматизировать разные процессы жизненного цикла сущностей. Теперь это возможно
не только с помощью мапперов: в Cycle ORM v2 добавлен компонент cycle/entity-macros, цель которого — упростить задачу
автоматизации.

В пакете `cycle/entity-macros` представлена альтернативная реализация генератора команд ORM, которая в процессе
формирования команд на сохранение/удаление сущностей также генерирует события (events) и использует предопределённые
слушатели (listeners) для их обработки.

### з коробки уже реализованы несколько базовых макросов:

- **UUID** - для генерации UUID. Необходимо установить дополнительный пакет cycle/entity-macros-uuid.
- **CreatedAt** и **UpdatedAt** — автоматически добавляет время создания и изменения записи.
- **DeletedAt (SoftDelete)** — при удалении записи в указанном поле выставляется метка времени удаления, а сама запись
  не удаляется.
- **OptimisticLock** — для предотвращения одновременного редактирования записи. При неудаче транзакция прерывается.
- **CallableSubscriber** — для вызова callback-функции.
- **ClassSubscriber** - для создания пользовательских макросов.

## Установка

```
composer require cycle/entity-macros
```

## Настройка

В приложении необходимо создать объект ORM используя класс `\Cycle\ORM\Entity\Macros\EventDrivenCommandGenerator`:

```php
$commandGenerator = new \Cycle\ORM\Entity\Macros\EventDrivenCommandGenerator($schema);
$orm = new \Cycle\ORM\ORM($factory, $schema, $commandGenerator); 
```

## UUID

Необходимо выполнить установку дополнительного пакета с UUID макросом:

```
composer require cycle/entity-macros-uuid dev-master
```

Макрос предназначен для генерации уникальных идентификаторов (UUID) в сущностях. Для использования доступны несколько
различных типов UUID.

### RFC 4122 UUIDs

#### Version 1: Time-based

A version 1 UUID uses the current time, along with the MAC address (or node) for a network interface on the local
machine.

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid1Macro;
use Ramsey\Uuid\UuidInterface;

#[Entity]
#[Uuid1Macro(field: 'uuid', node: '00000fffffff', clockSeq: 0xffff)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 2: DCE Security

UUID v2 uses the current time, along with the MAC address (or node) for a network interface on the local machine.
Additionally, a version 2 UUID replaces the low part of the time field with a local identifier such as the user ID or
group ID of the local account that created the UUID.

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid2Macro;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid2Macro(
    field: 'uuid',
    localDomain: Uuid::DCE_DOMAIN_PERSON, 
    localIdentifier: '12345678', 
    node: '00000fffffff', 
    clockSeq: 0xffff
)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 3: Name-based (MD5)

Uses a version 3 (name-based) UUID based on the MD5 hash of a namespace ID and a name

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid3Macro;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid3Macro(field: 'uuid', namespace: Uuid::NAMESPACE_URL, name: 'https://example.com/foo')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 4: Random

They are randomly-generated and do not contain any information about the time they are created or the machine that
generated them.

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid4Macro;
use Ramsey\Uuid\UuidInterface;

#[Entity]
#[Uuid4Macro]
class User
{
    #[Column(field: 'uuid', type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 5: Name-based (SHA-1)

Uses a version 5 (name-based) UUID based on the SHA-1 hash of a namespace ID and a name

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid5Macro;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid5Macro(field: 'uuid', namespace: Uuid::NAMESPACE_URL, name: 'https://example.com/foo')]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

#### Version 6: Nonstandard UUIDs

Ordered-Time Uses a version 6 (ordered-time) UUID from a host ID, sequence number, and the current time

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Uuid\Uuid6Macro;
use Ramsey\Uuid\UuidInterface;
use Ramsey\Uuid\Uuid;

#[Entity]
#[Uuid6Macro(field: 'uuid', node: '00000fffffff', clockSeq: 0xffff)]
class User
{
    #[Column(type: 'uuid', primary: true)]
    private UuidInterface $uuid;
}
```

Макрос UUID может сам объявить столбец uuid нужного типа и связать его со свойством $uuid сущности. Таким образом,
использование атрибута `#[Column]` не является обязательным, а параметры столбца, несовместимые с требованиями макроса,
приведут к исключению.

## OptimisticLock

Используется для предотвращения одновременного редактирования записи в базе данных. При блокировании записи транзакция
прерывается.

В макросе доступно несколько стратегий работы:

- `MICROTIME` - current timestamp with microseconds as string
- `RAND_STR` - случайно сгенерированная строка (random_bytes)
- `INCREMENT` - auto incremented integer version
- `DATETIME` - current datetime
- `CUSTOM` - позволяет контролировать и устанавливать версию самостоятельно. Необходимо вручную создавать поле в базе
  данных и в сущности и самостоятельно контролировать версию сущности.

#### Пример использования:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\OptimisticLock\OptimisticLockListener;
use Cycle\ORM\Entity\Macros\OptimisticLock\OptimisticLockMacro;

#[Entity]
#[OptimisticLockMacro(field: 'version', rule: OptimisticLockListener::RULE_INCREMENT)]
class Page
{
    #[Column(type: 'primary')]
    private int $id;
    
    #[Column(type: 'integer')]
    private int $version;
}
```

Можно не указывать параметр `field: 'version'` и не создавать свойство `version` в сущности, в таком случае, макрос
добавит поле `version` нужного типа автоматически.

## CreatedAt и UpdatedAt

Макросы предназначены для автоматического добавления даты создания и редактирования записи.

#### Пример использования:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Timestamped\CreatedAtMacro;
use Cycle\ORM\Entity\Macros\Timestamped\UpdatedAtMacro;

#[Entity]
#[CreatedAtMacro(field: 'createdAt', column: 'created_at')]
#[UpdatedAtMacro(field: 'updatedAt', column: 'updated_at')]
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

Макросы CreatedAtMacro и UpdatedAtMacro могут сами объявлять столбцы created_at и updated_at нужного типа и связывать их
со свойствами сущности. Таким образом, использование атрибута #[Column] не является обязательным, а параметры столбца,
несовместимые с требованиями макроса, приведут к исключению.

## DeletedAt (SoftDelete)

Реализовывает стратегию Soft Delete. При удалении сущности, команда на удаление заменяется на команду редактирования, в
поле deleted_at (конфигурируется в настройках макроса) добавляется дата удаления записи.

Пример использования:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Timestamped\DeletedAtMacro;

#[Entity]
#[DeletedAtMacro(field: 'deletedAt', column: 'deleted_at')]
class Page
{
    #[Column(type: 'primary')]
    public int $id;
        
    #[Column(type: 'datetime', nullable: true)]
    public ?\DateTimeImmutable $deletedAt = null;
}
```

## CallableSubscriber

Используется для вызова пользовательского `callable`, в котором можно выполнить необходимую вам операцию. В параметрах
необходимо передать callable, который необходимо вызвать и событие (или массив событий).

#### Пример использования:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Common\Event\Mapper\Command\OnCreate;
use Cycle\ORM\Entity\Macros\Subscriber\CallableSubscriberMacro;

#[Entity]
#[CallableSubscriberMacro([Comment::class, 'method'], events: OnCreate::class)]
class Comment
{
    #[Column(type: 'primary')]
    public int $id;
    
    public static function method(OnCreate $event): void
    {
        // some code
    }
}
```

#### Список доступных событий:

```
Cycle\ORM\Entity\Macros\Common\Event\Mapper\Command\OnCreate;
Cycle\ORM\Entity\Macros\Common\Event\Mapper\Command\OnUpdate;
Cycle\ORM\Entity\Macros\Common\Event\Mapper\Command\OnDelete
```

Можно подписываться сразу на несколько событий, например:

```php
#[CallableSubscriberMacro([Comment::class, 'method'], events: [OnCreate::class, OnUpdate::class])]
```

## Пользовательские макросы

Можно создавать собственные макросы, используя макрос `Cycle\ORM\Entity\Macros\Subscriber\ClassSubscriberMacro`. Он
позволяет добавить в схему Listener, в котором можно реализовать нужные методы и подписать их на нужные вам события.

#### Пример сущности:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Cycle\Annotated\Annotation\Column;
use Cycle\Annotated\Annotation\Entity;
use Cycle\ORM\Entity\Macros\Subscriber\ClassSubscriberMacro;
use App\Listener;

#[Entity]
#[ClassSubscriberMacro(CommentListener::class)]
class Comment
{
    #[Column(type: 'primary')]
    public int $id;
}
```

#### Пример класса слушателя:

```php
<?php

declare(strict_types=1);

namespace App\Listener;

use Cycle\ORM\Entity\Macros\Attribute\Listen;
use Cycle\ORM\Entity\Macros\Common\Event\Mapper\Command\OnCreate;

class CommentListener
{
    #[Listen(OnCreate::class)]
    public function method(OnCreate $event): void
    {
        // come code
    }
}
```

В конструкторе класса слушателя можно использовать зависимости из вашего DI.
