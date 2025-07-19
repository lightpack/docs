# Testing

> **Lightpack** extends `PHPUnit` to support integration tests for your HTTP routes, responses, and API endpoints. To feature test a route and its response, create a test class in `tests/Http` folder in your project root. 

You can see an example of a test class provided in `tests/Http` folder.

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

## Asserting HTML Responses

Once you have made a request to a route, you can then assert returned response using following assertion methods:

```php
$this->assertRouteNotFound();
$this->assertResponseStatus();
$this->assertResponseBody();
```

- **assertRouteNotFound()** 
  
Asserts that the last request resulted in a 404 Not Found response. Use this to verify that a route does not exist or is not accessible.

```php
$this->request('GET', '/invalid-route');
$this->assertRouteNotFound();
```

- **assertResponseStatus()** 

Asserts that the response status code matches the expected value. Pass the desired HTTP status code (e.g., 200 for OK, 302 for redirect, 404 for not found).

```php
$this->request('POST', '/login', [
    'email' => 'foo@bar.com', 
    'password' => 'secret'
]);

$this->assertResponseStatus(302);
```

- **assertResponseBody()**: 
  
Asserts that the response body matches the given string exactly. Use this to confirm the precise output returned by the route.

```php
$this->request('GET', '/hello');
$this->assertResponseBody('Hello, World!');
```

## Asserting JSON Responses

```php
$this->assertResponseHasValidJson();
$this->assertResponseJson();
$this->assertResponseJsonHasKey();
$this->assertResponseJsonKeyValue();
```

## Asserting Redirect Responses

```php
$this->assertRedirectUrl();
$this->assertResponseIsRedirect();
```

## Asserting Sessions

```php
$this->withSession();
$this->assertSessionHas();
$this->assertSessionHasErrors();
$this->assertSessionHasOldInput();
```

## Asserting Headers

```php
$this->withHeaders();
$this->assertResponseHasHeader();
$this->assertResponseHeaderEquals();
```


## Asserting Cookies

```php
$this->withCookies();
```

```php
public function testItSetsCookies()
{
    $this->withCookies(['test' => 'test']);
    $this->requestJson('get', '/api/test');

    $this->assertTrue(cookie()->has('test'));
    $this->assertEquals('test', cookie()->get('test'));
}
```                    

## Asserting File Uploads

```php
$this->withFiles();
```

## Asserting Mails

```php
$this->assertMailSent();
$this->assertMailNotSent();
$this->assertMailSubject();
$this->assertMailContains();
$this->assertMailSentFrom();
$this->assertMailCount();
$this->assertMailSentTo();
$this->assertNoMailSentTo();
$this->assertMailSentToAll();
$this->assertMailCc();
$this->assertMailBcc();
$this->assertMailCcAll();
$this->assertMailBccAll();
$this->assertMailReplyTo();
$this->assertMailReplyToAll();
$this->assertMailHasAttachments();
$this->assertMailHasAttachment();
$this->assertMailHasNoAttachments();
```

## Asserting Databases

**Lightpack** ships with `phpunit.xml` file in the project's root directory. You can specify the database configurations you will use for testing.

Once you have configured database settings, use the `Lightpack\Testing\DatabaseTrait` in your test classes that need to interect with database. 
- This migrates all the required schema changes per test method before it runs and destroys them after every test method execution completion.
- This also wraps the test method execution within a transaction ensuring a clean slate for each test method.

```php
class LoginTest extends TestCase
{
    use DatabaseTrait;

    public function testUserCanLoginWithValidData()
    {
        $user = [
            'email' => 'johndoe@example.com',
            'password' => '!secret!';
        ];

        // Make the login post request
        $this->request('POST', url()->route('login'), $user);

        // Assert tge response status is 302
        $this->assertResponseStatus(302);

        // Assert response is a redirect
        $this->assertInstanceOf(Redirect::class, $this->response);

        // Assert redirect url is to dashboard route
        $this->assertRedirectUrl(url()->route('dashboard'));

        // Assert user login session is set
        $this->assertTrue(auth()->isLoggedIn());
    }
}
```

### Seeding

In case you want to seed some test data, execute all your seeder classes:

```cli
php console db:seed
```