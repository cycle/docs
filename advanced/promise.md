# References and Proxies
The ORM provides the ability to reference to objects that are not currently loaded from the database. This can be achieved
using two main abstractions: Reference and Promises (proxies).

> Every proxy object is a reference. Note, the terms Proxy and Promise are used in ORM equally. The promise is responsible for the resolution of the remote entity Node on demand, while Proxy can also emulate an Entity as a real object. Every Proxy is a Promise, not every Promise is a Proxy.

## References
References provide the ability to replace an actual entity with a Reference object which points to the entity's role and it's scope (columns used for relations). In some cases, like in relations `belongsTo` and `refersTo`, the object would not be fetched from the database and the manually specified
scope will be used.

We can demonstrate it:

```php
/** @Entity */
class Post
{
    /** @Column(type="primary") */
    public $id;

    /** @BelongsTo(target="User") */
    public $user;
}
```

Now, if we want to create a new `Post` entity we have an option to set the user value as a `User` entity or use a reference instead:

```php
$p = new Post();
$p->user = new Reference('user', ['id' => 1]);

$t = new \Cycle\ORM\Transaction($orm);
$t->persist($p);
$t->run();
```

> You must use entity role in reference.

If you use references often it might be convenient to create custom object for such purpose:

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

$t = new \Cycle\ORM\Transaction($orm);
$t->persist($p);
$t->run();
```

## Proxies and Promises
More complex usages of references include the ability to gain access to a related entity or collection on demand
(without pre-loading data). By default ORM will wrap lazy loaded data using a Promise object, which does not extend the original
entity and cannot be used with strongly typed (PHP 7.4) entities. However, you can use an extension to generate a promise of each
desired entity.

You must install the Cycle Proxy Factory extension to do that:

```
$ composer require cycle/proxy-factory
```

And configure your orm instance:

```php
$orm = $orm->withPromiseFactory(new \Cycle\ORM\Promise\ProxyFactory());
```

Now all the lazy loaded objects will be accessed via a proxy:

```php
$post = $orm->getRepository(Post::class)->findOne();
print_r($post->user);
```

> Note, proxy usage applies a set of limitations on your entities. For example, you cannot declare your entity
as `final`, because the proxy has to extend it. Please check this article regarding 
[typical proxy limitations](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/limitations-and-known-issues.html#entities-proxies-and-reflection).
