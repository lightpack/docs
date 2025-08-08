# Events

Lightpack provides events support in very friendly manner. For example, 
sending account confirmation or payment due email both involve
utilizing email features. So instead of hard coding email service provider
reference in your controllers, you can notify your application components
to act on behalf of email related events.

<p class="tip">
You can use events to promote loose coupling and better code reuse of shareable
components.
</p>

## Defining Listener

To use events in Lighpack, first create a listener. For example, here we create 
a user event listener.

```php
<?php

namespace App\Events;

class UserCreatedEvent
{
    public function handle()
    {
        // ...
    }
}
```

## Registering Listener

Now register this listener in <code>boot/events.php</code> file.

```php
<?php

return [
    'user:created' => [
        App\Events\UserCreatedEvent::class,
    ]
];
```

## Notifying Listeners

Now anywhere you fire an event named <code>user:created</code>,
will call <code>App\Events\UserCreatedEvent::handle()</code> method.

```php
<?php

class UserController
{
    public function index()
    {
        event()->fire('user:created');
    }
}
```

## Passing Event Data

You can pass any data as second argument to `fire()` method.

```php
event()->fire('user:created', $user);
```

Now you can access the event data array in the `handle()` method of the listener.

```php
public function handle(User $user)
{
    // ...
}
```