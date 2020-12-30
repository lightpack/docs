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
a UI menu event listener.

```php
<?php

namespace App\Events;

class MenuEventListener
{
    public function handle()
    {
        app('event')->setData([
            '/' => 'Home', 
            '/about' => 'About', 
            '/contact' => 'Contact'
        ]);
    }
}
```

## Configuring Listener

Now configure this listener in <code>config/events.php</code> file.

```php
<?php

return [
    'menu::render-before' => [
        App\Events\MenuEventListener::class,
    ]
];
```

## Notifying Listeners

Now anywhere you fire an event named <code>menu::render-before</code>,
will call <code>App\Events\MenuEventListener::handle()</code> method.

```php
<?php

class PageController
{
    public function index()
    {
        app('event')->notify('menu::render-before');
        $menu = app('event')->getData();
    }
}
```