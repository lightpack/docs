# Routing

Routing is the process of mapping requested URLs with appropriate
request handlers aka <code>controllers</code>.

```php
$route->get('/products', ProductController::class);
```

All your routes definitions goes in file: <code>config/routes.php</code> 

## Route Methods

Lightpack supports following routing methods for your convinience:
* <code>$route->get()</code>
* <code>$route->post()</code>
* <code>$route->put()</code>
* <code>$route->patch()</code>
* <code>$route->delete()</code>
* <code>$route->any()</code>
* <code>$route->map()</code>
* <code>$route->group()</code>

## Route Parameters

Lightpack supports regex based route paths definition. It comes with a bunch of
regex patterns out of the box to assist you for common cases.

* <code>:any</code>
* <code>:num</code>
* <code>:seg</code>
* <code>:slug</code>
* <code>:alpha</code>
* <code>:alnum</code>

```php
// matches path: /users/23
$route->get('/users/:num', UserController::class, 'find');

// matches path: /users/23/status/active
$route->get('/users/:num/status/:str', UserController::class, 'filter');
```

Below is an explanation for pre-defined regex route placeholders.

<table>
    <tbody>
        <tr>
            <td><code>:any</code></td>
            <td>Match any and all the characters in the URI. Equivalent regex pattern <code>(.*)</code></td>
        </tr>
        <tr>
            <td><code>:num</code></td>
            <td>Match a numeric segment in the URI. Equivalent regex pattern <code>([0-9]+)</code></td>
        </tr>
        <tr>
            <td><code>:seg</code></td>
            <td>Match a segment in the URI. Equivalent regex pattern <code>([^/]+)</code></td>
        </tr>
        <tr>
            <td><code>:slug</code></td>
            <td>Match a slug based segment in the URI. Equivalent regex pattern <code>([a-zA-Z0-9-]+)</code></td>
        </tr>
        <tr>
            <td><code>:alpha</code></td>
            <td>Match a segment of alphabet characters in the URI. Equivalent regex pattern <code>([a-zA-Z]+)</code></td>
        </tr>
        <tr>
            <td><code>:alnum</code></td>
            <td>Match an alphanumeric segment in the URI. Equivalent regex pattern <code>([a-zA-Z0-9]+)</code></td>
        </tr>
    </tbody>
</table>

## Route Regex

You can provide your own custom regular expression to match the path.

```php
$route->get('/users/id/([0-9]{4})', UserController::class, 'findById');
```                

## Route Prefix

If you have a bunch of routes that start with common path prefix,
you can group them using <code>$route->group()</code> method.

For example, the following route group definition will match
request paths that start with <code>/api/v1</code>.

```php
$route->group(['prefix' => '/api/v1'], function($route) {
    $route->get('/users', UserController::class, 'list');
});
```