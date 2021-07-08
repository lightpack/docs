# Containers

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

## Register Instance

In some cases you might want a new instance every time you access the service through
the container. For that use the <code>factory()</code> method of container.

```php
$container->factory('service', function($container) {
    return new ServiceProvider();
});
```

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
