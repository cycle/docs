# References and Proxies
ORM provides the ability to use and reference to the objects which are not currently loaded from the database. Such thing can be achieved
using two main abstractions Reference and Promises (proxies).

> Every proxy object is a reference. Note, the termin Proxy and Promise are used in ORM equally. Promise is responsible for the resolution of remote entity Node on demand, while Proxy can also emulate Entity as real object. Every Proxy is a Promise, not every Promise is Proxy.

## References
References provide the ability to replace actual entity with Reference object which points to entity role and it's scope (columns used 
for relations). In some case, like in relations `belongsTo` and `refersTo` object would not be fetched from database and manually specified
scope will be used. 

We can demonstrate it:

```php
/** @Entity */
class Post
{
    /** @Column(type=primary) */
    public $id;

    /** @BelongsTo(target=User) */
    public $user;
}
```

Now, if we want to create new `Post` entity we have an option to set user value as `User` entity or use reference instead:

```php
$p = new Post();
$p->user = new Reference('user', ['id' => 1]);

$t = new Transaction($orm);
$t->persist($p);
$t->run();
```

> You must use entity role in reference.

If you use references often it might be convinient to create custom object for such purpose:

```php
use Cycle\ORM\Promise\ReferenceInterface;

class UserID implements ReferenceInterface
{
    private $id;

    public function __construct($id)
    {
        $this->id = $id;
    }

    public function __role(): string
    {
        return 'user';
    }

    public function __scope(): array
    {
        return ['id' => $this->id];
    }
}
```

And use it accordingly:

```php
$p = new Post();
$p->user = new UserID(1);

$t = new Transaction($orm);
$t->persist($p);
$t->run();
```

## Proxies and Promises
More complex variation of references include the ability to gain access to related entity or collection on demand
(without pre-loading data). By default ORM will wrap lazy loaded data using Promise object which does not extend original
entity and can not be used with strongly typed entities. However, you can use an extension to generate promise of each
desired entity.

You must connect Cycle extension to do that:

```
$ composer require cycle/proxy-factory
```

And configure your orm intance:

```php
$orm = $orm->withPromiseFactory(new \Cycle\ORM\Promise\ProxyFactory());
```

Now all the lazy loaded objects will be accessed via proxy:

```php
$post = $orm->getRepository(Post::class)->findOne();
print_r($post->user);
```

> Note, proxy usage applies set of limitations on your entities. You can not use `final` statement as your class won't be extended. 
Please check this article regarding [typical proxy limitations](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/limitations-and-known-issues.html).
