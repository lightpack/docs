# Testing

> **Lightpack** extends `PHPUnit` to support integration tests for your HTTP routes, responses, and API endpoints. 

To feature test a route and its response, create a test class in `tests` folder in your project root. You can see
an example of a test class provided in `tests/Http` folder.

```php
class HomeTest extends HttpTestCase
{
    public function testItRendersHomePage()
    {
        $response = $this->request('GET', '/');

        $this->assertResponseStatus(200);
    }
}
```

## Testing Routes

To test an HTTP route, use `request()` method. For testing a **JSON** API route, use `requestJson()` method. Both of these methods are used to simulate an HTTP request.

### request()

This method takes 3 parameters:

* Request method,
* HTTP route,
* Optional payload.

For example, to make a `GET` request to `/homepage` route:

```php
$this->request('GET', '/homepage');
```

For example, to make a `POST` request to `/products` route:

```php
$this->request('POST', '/products', ['name' => 'Lightpack']);
```

### requestJson()

Use this method to make **JSON** requests. It accepts the same parameters that `request()` method does. For example, to make a **JSON** `POST` request to `/products` route:

```php
$this->requestJson('POST', '/products', ['name' => 'Lightpack']);
```

## Asserting Responses

Once you have made a request to a route, you can then assert returned response using following assertion methods:

```php
$this->assertResponseStatus();
$this->assertResponseBody();
$this->assertResponseHasValidJson();
$this->assertResponseJson();
$this->assertResponseJsonHasKey();
$this->assertResponseJsonKeyValue();
$this->assertResponseHasHeader();
$this->assertRedirectUrl();
$this->assertResponseHeaderEquals();
$this->assertRouteNotFound();
```

### assertResponseStatus()

Use this method to assert the returned response status code. For example:

```php
public function testItRendersHomePage()
{
    $this->request('GET', '/');

    $this->assertResponseStatus(200);
}
```

### assertResponseBody()

Use this method to assert the returned content in response body. For example:

```php
public function testItRendersHomePage()
{
    $this->request('GET', '/');

    $this->assertResponseBody('<h1>Welcome</h1>');
}
```