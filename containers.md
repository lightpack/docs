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

### app() Helper

The `app()` helper function provides convenient access to the container and its services.

**Get the container instance:**

```php
$container = app();
```

**Get a registered service:**

```php
$mailer = app('mailer');
```

**Auto-resolve a class:**

```php
// Automatically resolves dependencies
$service = app(UserService::class);
```

The `app()` function intelligently handles three scenarios:
- **No argument** → Returns the container instance
- **Registered service** → Returns the service via `get()`
- **Unregistered class** → Auto-resolves via `resolve()`

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

### call()

Resolve and call a method with automatic dependency injection:

```php
class UserService
{
    public function sendEmail(Mailer $mailer, string $to, string $subject)
    {
        // $mailer auto-injected, $to and $subject from args
    }
}

// Call method with dependency injection
$container->call(UserService::class, 'sendEmail', [
    'to' => 'user@example.com',
    'subject' => 'Welcome'
]);

// Or with an instance
$service = new UserService();
$container->call($service, 'sendEmail', ['to' => 'user@example.com', 'subject' => 'Welcome']);
```

The container resolves type-hinted dependencies and merges them with provided scalar arguments.

### callIf()

Safely call a method only if it exists:

```php
// Won't throw exception if method doesn't exist
$container->callIf($service, 'boot');
```

Useful for optional lifecycle hooks or conditional method calls.

### reset()

Clear all registered services, bindings, and aliases:

```php
$container->reset();
```

Useful for testing when you need a clean container state.

---

## Lifecycle Hooks

### __boot()

The container automatically calls `__boot()` method on resolved services if it exists:

```php
class DatabaseService
{
    public function __construct(Config $config)
    {
        // Constructor runs first
    }

    public function __boot()
    {
        // Called automatically after construction
        // Useful for initialization that needs the service fully constructed
    }
}

// Container automatically calls __boot() after construction
$db = $container->resolve(DatabaseService::class);
```

This hook is called for:
- Services resolved via `resolve()`
- Services registered via `register()` (on first access)

The hook is **optional** - only implement it if you need post-construction initialization.

---

### getInstance()

The container itself is a singleton. You can get the instance anywhere:

```php
$container = Container::getInstance();
```

For testing purposes, you can destroy the instance:

```php
Container::destroy();
```
