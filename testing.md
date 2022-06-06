# Testing

> **Lightpack** extends `PHPUnit` to support integration tests for your HTTP routes, responses, and API endpoints. 

To feature test a route and its response, create a test class in `tests` folder in your project root. You can see
an example of a test class provided in `tests/Http` folder.

```php
class HomeTest extends HttpTestCase
{
    public function testItRendersHomePage()
    {
        $this->request('GET', '/');
        $this->assertResponseStatus(200);
    }
}
```

## Making Requests

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
$this->assertRouteNotFound();
$this->assertResponseStatus();
$this->assertResponseBody();
$this->assertResponseHasValidJson();
$this->assertResponseJson();
$this->assertResponseJsonHasKey();
$this->assertResponseJsonKeyValue();
$this->assertResponseHasHeader();
$this->assertResponseHeaderEquals();
$this->assertRedirectUrl();
```

### assertRouteNotFound()

Use this method to assert that request route does not exist.

```php
public function testPageNotFound()
{
    $this->request('GET', '/does-not-exist');

    $this->assertRouteNotFound();
}
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

### assertResponseHasValidJson()

Use this method to assert that returned **JSON** in response is valid. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products');

    $this->assertResponseHasValidJson();
}
```

### assertResponseJson()

Use this method to assert that returned **JSON** in response exactly matches the array. For example:


```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseJson(['name' => 'Lightpack']);
}
```

### assertResponseJsonHasKey()

Use this method to assert that returned **JSON** in response contains a given key. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseJsonHasKey('name');
}
```

Note that you can also specify a key using `dot` notation. For example:


```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseJsonHasKey('brand.name');
}
```

### assertResponseJsonKeyValue()

Use this method to assert that returned **JSON** in response has matching `key-value`. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseJsonHasKey('name', 'Lightpack');
}
```

### assertResponseHasHeader()

Use this method to assert that returned response has matching header key. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseHasHeader('cache-control');
}
```

### assertResponseHeaderEquals()

Use this method to assert that returned response has matching header `key-value`. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/products/1');

    $this->assertResponseHasHeader('cache-control', 'no-cache');
}
```

### assertRedirectUrl()

Use this method to assert that returned response has a redirect URL. For example:

```php
public function testItFetchesProductsJson()
{
    $this->request('GET', '/dashboard');

    $this->assertResponseHasHeader('/login');
}
```

## Testing Databases

> Note: This section applies only in case of MariaDB, MySQl based relational databases.

**Lightpack** ships with `phpunit.xml` file in the project's root directory. You can specify the database name you will use for testing:

```xml
<env name="DB_NAME" value="test_lightpack" />
```

### Migrations

Before you start testing database application, you should run all your migrations in the test database so that it contains upto date schema for testing:

```cli
php lucy migrate:up
```

### Seeding

In case you want to seed some test data, execute all your seeder classes:

```cli
php lucy db:seed
```

### Transactions

You should wrap your test methods in transactions so that once the method is tested, it should rollback any changes made in database. For that, simply wrap your statements within a transaction:

```php
public function testItStoresNewProduct()
{
    app('db')->begin();

    $this->request('POST', '/products', ['name' => 'Lightpack']);
    
    $this->assertResponseStatus(201);
    
    app('db')->rollback();
}
```