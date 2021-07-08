# Filters

Filters act as hooks that work with controllers. Filters get executed just before or after 
a controller action is executed.

You can configure filters while defining app routes.

```php
$route->get('/dashboard', DashboardController::class, 'index', ['auth']);
```

You can also group filters as:

```php
$route->group(['filters' => ['csrf', 'auth']], function($route) {
    $route->get('/dashboard', DashboardController::class, 'index');
});
```

<p class="tip">
To halt the request early even before executing the target controller's action, simply
return a response.
</p>

<p class="tip">
You can modify the controller's action response in the filters <code>after()</code> method.
</p>

## Defining Filter

Filters are classes that implement <code>IFilter</code> interface. They are defined
in <code>app/filters</code> folder.

```php
<?php

use Lightpack\Http\Request;
use Lightpack\Http\Response;
use Lightpack\Filters\IFilter;

class TrimFilter implements IFilter
{
    public function before(Request $request)
    {
        // ...
    }

    public function after(Request $request, Response $response): Response
    {
        // ...
    }
}
```

## Configuring Filter

Once you define a filter class, you must configure it in <code>config/filters.php</code>.
Simply provide a filter <code>alias</code> as key and filter class fully quilified 
<code>namespace</code> as value.

```php
<?php

return [
    'trim' => App\Filters\TrimFilter::class,
];
```            