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

- **assertResponseHasValidJson()**
  
  Asserts that the response body is valid JSON. Use this to ensure your endpoint returns a properly formatted JSON response.

  ```php
  $this->request('GET', '/api/products');
  $this->assertResponseHasValidJson();
  ```

- **assertResponseJson()**
  
  Asserts that the response JSON matches the given array exactly. Use this to check for a full match with the expected JSON structure and values.

  ```php
  $this->request('GET', '/api/products/1');
  $this->assertResponseJson(['id' => 1, 'name' => 'Lightpack']);
  ```

- **assertResponseJsonHasKey()**
  
  Asserts that a specific key exists in the response JSON. Supports dot notation for nested keys.

  ```php
  $this->request('GET', '/api/user');
  $this->assertResponseJsonHasKey('profile.email');
  ```

- **assertResponseJsonKeyValue()**
  
  Asserts that a specific key in the JSON response has the expected value. Supports dot notation for nested keys.

  ```php
  $this->request('GET', '/api/user');
  $this->assertResponseJsonKeyValue('profile.email', 'john@example.com');
  ```

## Asserting Redirect Responses

```php
$this->assertRedirectUrl();
$this->assertResponseIsRedirect();
```

- **assertRedirectUrl()**
  
  Asserts that the response is a redirect to the specified URL. Use this to confirm the correct redirect location after an action.

  ```php
  $this->request('POST', '/login', [
      'email' => 'foo@bar.com',
      'password' => 'secret'
  ]);
  $this->assertRedirectUrl('/dashboard');
  ```

- **assertResponseIsRedirect()**
  
  Asserts that the response is a redirect (any 3xx status code). Use this to check if the action results in a redirect, regardless of destination.

  ```php
  $this->request('POST', '/logout');
  $this->assertResponseIsRedirect();
  ```

## Asserting Sessions

```php
$this->withSession();
$this->assertSessionHas();
$this->assertSessionHasErrors();
$this->assertSessionHasOldInput();
```

- **withSession()**
  
  Sets session data before making a request. Use this to simulate a session state, such as a logged-in user or pre-filled data.

  ```php
  $this->withSession(['user_id' => 1]);
  $this->request('GET', '/dashboard');
  ```

- **assertSessionHas()**
  
  Asserts that the session contains the given key (and optionally, the expected value). Use this to check if session data is set after a request.

  ```php
  $this->request('POST', '/login', [
      'email' => 'foo@bar.com',
      'password' => 'secret'
  ]);
  $this->assertSessionHas('user_id');
  ```

- **assertSessionHasErrors()**
  
  Asserts that the session contains validation errors for the given fields. Use this to verify validation error handling.

  ```php
  $this->request('POST', '/register', [
      'email' => '',
      'password' => ''
  ]);
  $this->assertSessionHasErrors(['email', 'password']);
  ```

- **assertSessionHasOldInput()**
  
  Asserts that the session contains old input data for the specified fields. Useful for checking form repopulation after validation errors.

  ```php
  $this->request('POST', '/register', [
      'email' => 'foo@bar.com',
      'password' => ''
  ]);
  $this->assertSessionHasOldInput(['email']);
  ```

## Asserting Headers

```php
$this->withHeaders();
$this->assertResponseHasHeader();
$this->assertResponseHeaderEquals();
```

- **withHeaders()**
  
  Sets custom HTTP headers for the next request. Use this to simulate requests with specific headers (e.g., authentication, content type).

  ```php
  $this->withHeaders([
      'Authorization' => 'Bearer token',
      'Accept' => 'application/json'
  ]);
  $this->request('GET', '/api/data');
  ```

- **assertResponseHasHeader()**
  
  Asserts that the response contains the specified header. Use this to check if a header is present in the response.

  ```php
  $this->request('GET', '/api/data');
  $this->assertResponseHasHeader('Content-Type');
  ```

- **assertResponseHeaderEquals()**
  
  Asserts that a response header matches the expected value. Use this to verify the exact value of a header in the response.

  ```php
  $this->request('GET', '/api/data');
  $this->assertResponseHeaderEquals('Content-Type', 'application/json');
  ```


## Asserting Cookies

```php
$this->withCookies();
$this->assertCookieHas();
$this->assertCookieEquals();
$this->assertCookieMissing();
```

- **withCookies()**
  
  Sets cookies for the next request. Use this to simulate requests with specific cookie values (e.g., authentication, preferences).

  ```php
  $this->withCookies(['theme' => 'dark', 'token' => 'abc123']);
  $this->request('GET', '/profile');
  ```

- **assertCookieHas()**
  
  Asserts that a cookie exists in the response.

  ```php
  $this->request('GET', '/set-cookie');
  $this->assertCookieHas('theme');
  ```

- **assertCookieEquals()**
  
  Asserts that a cookie exists and has the expected value.

  ```php
  $this->request('GET', '/set-cookie');
  $this->assertCookieEquals('theme', 'dark');
  ```

- **assertCookieMissing()**
  
  Asserts that a cookie does not exist in the response.

  ```php
  $this->request('GET', '/set-cookie');
  $this->assertCookieMissing('old_token');
  ```

## Asserting File Uploads

```php
$this->withFiles();
```

- **withFiles()**
  
  Sets files for the next request. Use this to simulate file uploads in your tests.

  ```php
  $file = [
      'name' => 'avatar.png',
      'type' => 'image/png',
      'tmp_name' => __DIR__ . '/fixtures/avatar.png',
      'error' => UPLOAD_ERR_OK,
      'size' => filesize(__DIR__ . '/fixtures/avatar.png'),
  ];
  $this->withFiles(['avatar' => $file]);
  $this->request('POST', '/upload');
  ```

## Asserting File Uploads

- **assertResponseStatus()** (after upload)
  
  Asserts that the upload response status is as expected.

  ```php
  $this->request('POST', '/upload', [], ['avatar' => $file]);
  $this->assertResponseStatus(200);
  ```

- **assertTrue() / assertEquals()** (for uploaded file checks)
  
  Use these to check if the file exists in storage or matches expected properties after upload.

  ```php
  $this->request('POST', '/upload', [], ['avatar' => $file]);
  $this->assertTrue(file_exists(storage_path('uploads/avatar.png')));
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


- **assertMailSent()**
  
  Asserts that at least one email was sent during the test.
  ```php
  $this->assertMailSent();
  ```

- **assertMailNotSent()**
  
  Asserts that no emails were sent during the test.
  ```php
  $this->assertMailNotSent();
  ```

- **assertMailSubject()**
  
  Asserts that an email with the given subject was sent.
  ```php
  $this->assertMailSubject('Welcome to Lightpack');
  ```

- **assertMailContains()**
  
  Asserts that the email body contains the given text.
  ```php
  $this->assertMailContains('Your verification code is');
  ```

- **assertMailSentFrom()**
  
  Asserts that an email was sent from the given address.
  ```php
  $this->assertMailSentFrom('noreply@example.com');
  ```

- **assertMailCount()**
  
  Asserts that the number of emails sent matches the expected count.
  ```php
  $this->assertMailCount(2);
  ```

- **assertMailSentTo()**
  
  Asserts that an email was sent to the given recipient.
  ```php
  $this->assertMailSentTo('user@example.com');
  ```

- **assertNoMailSentTo()**
  
  Asserts that no email was sent to the given recipient.
  ```php
  $this->assertNoMailSentTo('banned@example.com');
  ```

- **assertMailSentToAll()**
  
  Asserts that emails were sent to all recipients in the given list.
  ```php
  $this->assertMailSentToAll(['a@example.com', 'b@example.com']);
  ```

- **assertMailCc()**
  
  Asserts that an email was CC'd to the given address.
  ```php
  $this->assertMailCc('manager@example.com');
  ```

- **assertMailBcc()**
  
  Asserts that an email was BCC'd to the given address.
  ```php
  $this->assertMailBcc('auditor@example.com');
  ```

- **assertMailCcAll()**
  
  Asserts that all given addresses were CC'd on the email.
  ```php
  $this->assertMailCcAll(['a@example.com', 'b@example.com']);
  ```

- **assertMailBccAll()**
  
  Asserts that all given addresses were BCC'd on the email.
  ```php
  $this->assertMailBccAll(['a@example.com', 'b@example.com']);
  ```

- **assertMailReplyTo()**
  
  Asserts that the email has the given reply-to address.
  ```php
  $this->assertMailReplyTo('support@example.com');
  ```

- **assertMailReplyToAll()**
  
  Asserts that all given reply-to addresses are set on the email.
  ```php
  $this->assertMailReplyToAll(['a@example.com', 'b@example.com']);
  ```

- **assertMailHasAttachments()**
  
  Asserts that the email has at least one attachment.
  ```php
  $this->assertMailHasAttachments();
  ```

- **assertMailHasAttachment()**
  
  Asserts that the email has the specified attachment.
  ```php
  $this->assertMailHasAttachment('invoice.pdf');
  ```

- **assertMailHasNoAttachments()**
  
  Asserts that the email has no attachments.
  ```php
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