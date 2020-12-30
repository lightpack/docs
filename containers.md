# Containers

Lightpack provides an IoC container to configure your class dependencies in a manner that
helps write maintainable code. 

<p class="tip">
IoC refers to the concept of inversion of control where dependencies are injected
into a class rather than hard coding into it.
</p>

## Register Singleton

Lightpack provides <code>config/services.php</code> file where you will see some 
services already registered into the container.

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

To access a registered service through the container, use <code>app('service')</code>
utility function. 

```php
app('service');
```