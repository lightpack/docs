# Routing

Routing is the process of mapping requested URLs with appropriate
request handlers called **controllers**. You can define your routes using `route()` function which returns 
an instance of route object provided by **Lightpack**.

```php
route()->get('/products', ProductController::class);
```

This will map the URL `/products` to the `ProductController` class and the `index` method by default. To map a different method, pass the method name as the last parameter.

```php
route()->get('/products', ProductController::class, 'list');
```

## Route Files

All the routes definitions goes into `routes` folder. There you will find two routes files already defined.

```text
routes
  ├── web.php
  ├── api.php
```

If you are defining routes for your APIs, you should define them in `routes/api.php` file.

## Route Methods

Lightpack supports following routing methods for your convinience:
* <code>route()->get()</code>
* <code>route()->post()</code>
* <code>route()->put()</code>
* <code>route()->patch()</code>
* <code>route()->delete()</code>
* <code>route()->options()</code>
* <code>route()->any()</code>
* <code>route()->map()</code>
* <code>route()->group()</code>

## Route Parameters

You can define dynamic parameters in your routes. For example, if you want to define a route for a product with id 1, you can define it as follows:

```php
route()->get('/products/:id', ProductController::class);
```

## Route Regex

Lightpack supports regex based route paths definition. It comes with a bunch of
regex patterns out of the box to assist you for common cases.

* <code>:any</code>
* <code>:num</code>
* <code>:seg</code>
* <code>:slug</code>
* <code>:alpha</code>
* <code>:alnum</code>

```php
// Example: /users/23
route()->get('/users/:user', UserController::class)->pattern(['user' => ':num'])
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

You can provide your own custom regular expression to match the path.

```php
route()->get('/users/:id', UserController::class)->pattern(['id' => '([0-9]{4})']);
```   

## Route Names

You can define a name for your route. This is useful when you want to generate a URL for a route.

```php
route()->get('/products/:id', ProductController::class)->name('product.show');
```

You can generate a URL for a route by passing the route name and parameters using the `url()` helper function. For example, if you want to generate a URL for the above route, you can do it as follows:

```php
url()->route('product.show', 1]);
```

## Route Filters

You can apply filters on a route by passing a `string` or an `array` of filters aliases as last argument.

```php
route()->get('/users', UserController::class)->filter('auth');
route()->post('/users', UserController::class)->filter(['auth', 'csrf']);
```

See more about filters [here](https://lightpack.github.io/docs/#/filters)

## Route Groups

If you have a bunch of routes that start with common path prefix, or apply common filters,
you can group them using <code>route()->group()</code> method.

This method takes an array as first argument and a callback function.

For example, the following route group definition will match
request paths that start with <code>/api/v1</code>.

```php
route()->group(['prefix' => '/api/v1'], function() {
    route()->get('/users', UserController::class);
});
```

Similarly, you can also group filters.

```php
route()->group(['filter' => ['cors', 'trim']], function() {
    route()->get('/users', UserController::class);
});
```