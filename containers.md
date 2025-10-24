# Container

Lightpack provides an IoC container to configure your class dependencies in a manner that
helps write maintainable code. 

<p class="tip">
IoC refers to the concept of inversion of control where dependencies are injected
into a class rather than hard coding into it.
</p>

Note that **container** in `Lightpack` is a lightweight [service locator](https://en.wikipedia.org/wiki/Service_locator_pattern).

<p class="tip">That means, it doesn't inject dependencies, it locates them for you.</p>

## Register Singleton

To bind an instance of a class just once, call `register()` method.

It takes an **alias** as string and a callback where you configure and return your class
instance.

```php
$container->register('service', function($container) {
    return new ServiceProvider();
});
```

The <code>register()</code> method essentially ends up creating a singleton of the
registered service.

<p class="tip">
By default, all the services are registered in a lazy manner.
</p>

## Register Factory

In some cases you might want a **new instance** every time you access the service through
the container. For that use the <code>factory()</code> method of container.

```php
$container->factory('service', function($container) {
    return new ServiceProvider();
});
```

<p class="tip">Use <code>factory()</code> when you need a fresh instance each time, and <code>register()</code> when you want a singleton.</p>

## Register Instance

You can register an already created instance directly:

```php
$mailer = new Mailer('smtp.example.org', 25);
$mailer->setUsername('your username');
$mailer->setPassword('your password');

// Register the configured instance
$container->instance('mailer', $mailer);
```

This is useful when you need to configure an object before registering it in the container.

## Accessing Service

To access a registered service through the container, use <code>app()</code>
utility function passing it the **alias** of the service you want to access from the container. 

```php
app('service');
```

## Other Methods

There are few more methods that you should know about.

### has()

To check if a **service/class** is registered in container, call `has()` method
passing it the **alias**. This method return a `boolean` value.

```php
$container->has('service'); // true
```

### get()

To get a configured item from the **container**, call `get()` method passing it the **alias**.

```php
$container->get('service');
```

### resolve()

The container can automatically resolve classes with dependencies using reflection. This is one of the most powerful features.

```php
class UserService
{
    public function __construct(Database $db, Cache $cache)
    {
        // Dependencies auto-injected
    }
}

// Container resolves dependencies automatically
$service = $container->resolve(UserService::class);
```

<p class="tip">The <code>resolve()</code> method works for any class with type-hinted constructor dependencies that are registered in the container.</p>

### bind()

Bind an interface or abstract class to a concrete implementation:

```php
$container->bind(MailerInterface::class, SmtpMailer::class);

// Resolve interface
$mailer = $container->resolve(MailerInterface::class);  // Gets SmtpMailer
```

This is useful for dependency inversion and testing.

### alias()

Create an alias for a service to enable multiple ways of accessing it:

```php
$container->register('auth', function() {
    return new Auth();
});

// Create alias for class name
$container->alias(Auth::class, 'auth');

// Now both work:
app('auth');           // Via alias
app(Auth::class);      // Via class name
```

### getInstance()

The container itself is a singleton. You can get the instance anywhere:

```php
$container = Container::getInstance();
```

For testing purposes, you can destroy the instance:

```php
Container::destroy();
```
