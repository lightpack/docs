# Testing

> **Lightpack** extends `PHPUnit` to support integration tests for your HTTP routes, responses, and API endpoints. To feature test a route and its response, create a test class in `tests/Http` folder in your project root. 

You can see an example of a test class provided in `tests/` folder.

```php
<?php

namespace Tests\Http;

use Lightpack\Testing\TestCase;

class HomeTest extends TestCase
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

## Asserting Emails

```php
$this->assertMailSent();
$this->assertMailNotSent();
$this->assertMailCount();
$this->assertMailSubject();
$this->assertMailContains();
$this->assertMailSentFrom();
$this->assertMailSentTo();
$this->assertNoMailSentTo();
$this->assertMailSentToAll();
$this->assertMailCc();
$this->assertMailCcAll();
$this->assertMailBcc();
$this->assertMailBccAll();
$this->assertMailReplyTo();
$this->assertMailReplyToAll();
$this->assertMailHasAttachment();
$this->assertMailHasAttachments();
$this->assertMailHasNoAttachments();
```

- **assertMailSent()**
  
  Asserts that at least one email was sent during the test.

  ```php
  $this->request('POST', '/register', ['email' => 'new@example.com']);
  $this->assertMailSent();
  ```

- **assertMailNotSent()**
  
  Asserts that no emails were sent during the test.

  ```php
  $this->request('GET', '/');
  $this->assertMailNotSent();
  ```

- **assertMailCount()**
  
  Asserts that an exact number of emails were sent.

  ```php
  $this->request('POST', '/invite-team', [
      'emails' => ['user1@example.com', 'user2@example.com']
  ]);
  $this->assertMailCount(2);
  ```

- **assertMailSubject()**
  
  Asserts that an email with the specified subject was sent.

  ```php
  $this->request('POST', '/register', ['email' => 'new@example.com']);
  $this->assertMailSubject('Welcome to Lightpack!');
  ```

- **assertMailContains()**
  
  Asserts that an email body contains the specified text.

  ```php
  $this->request('POST', '/register', ['email' => 'new@example.com']);
  $this->assertMailContains('Thank you for registering');
  ```

- **assertMailSentFrom()**
  
  Asserts that an email was sent from the specified address.

  ```php
  $this->request('POST', '/contact', ['message' => 'Hello']);
  $this->assertMailSentFrom('noreply@example.com');
  ```

- **assertMailSentTo()**
  
  Asserts that an email was sent to the specified recipient.

  ```php
  $this->request('POST', '/register', ['email' => 'new@example.com']);
  $this->assertMailSentTo('new@example.com');
  ```

- **assertNoMailSentTo()**
  
  Asserts that no email was sent to the specified recipient.

  ```php
  $this->request('POST', '/register', ['email' => 'new@example.com']);
  $this->assertNoMailSentTo('admin@example.com');
  ```

- **assertMailSentToAll()**
  
  Asserts that an email was sent to all specified recipients.

  ```php
  $this->request('POST', '/invite-team', [
      'emails' => ['user1@example.com', 'user2@example.com']
  ]);
  $this->assertMailSentToAll(['user1@example.com', 'user2@example.com']);
  ```

- **assertMailCc()**
  
  Asserts that an email was CC'd to the specified address.

  ```php
  $this->request('POST', '/send-report');
  $this->assertMailCc('manager@example.com');
  ```

- **assertMailCcAll()**
  
  Asserts that an email was CC'd to all specified addresses.

  ```php
  $this->request('POST', '/send-report');
  $this->assertMailCcAll(['manager@example.com', 'supervisor@example.com']);
  ```

- **assertMailBcc()**
  
  Asserts that an email was BCC'd to the specified address.

  ```php
  $this->request('POST', '/send-invoice');
  $this->assertMailBcc('accounting@example.com');
  ```

- **assertMailBccAll()**
  
  Asserts that an email was BCC'd to all specified addresses.

  ```php
  $this->request('POST', '/send-invoice');
  $this->assertMailBccAll(['accounting@example.com', 'archive@example.com']);
  ```

- **assertMailReplyTo()**
  
  Asserts that an email has the specified reply-to address.

  ```php
  $this->request('POST', '/contact', ['email' => 'customer@example.com']);
  $this->assertMailReplyTo('support@example.com');
  ```

- **assertMailReplyToAll()**
  
  Asserts that an email has all specified reply-to addresses.

  ```php
  $this->request('POST', '/contact');
  $this->assertMailReplyToAll(['support@example.com', 'sales@example.com']);
  ```

- **assertMailHasAttachment()**
  
  Asserts that an email has the specified attachment.

  ```php
  $this->request('POST', '/send-invoice');
  $this->assertMailHasAttachment('invoice.pdf');
  ```

- **assertMailHasAttachments()**
  
  Asserts that an email has all specified attachments.

  ```php
  $this->request('POST', '/send-documents');
  $this->assertMailHasAttachments(['contract.pdf', 'terms.pdf']);
  ```

- **assertMailHasNoAttachments()**
  
  Asserts that an email has no attachments.

  ```php
  $this->request('POST', '/send-notification');
  $this->assertMailHasNoAttachments();
  ```

## Database Testing

For tests that interact with the database, use the `DatabaseTrait` to automatically manage migrations and transactions.

```php
<?php

use Lightpack\Testing\TestCase;
use Lightpack\Testing\DatabaseTrait;

class UserDatabaseTest extends TestCase
{
    use DatabaseTrait;
    
    public function testUserCreation()
    {
        $user = new User();
        $user->email = 'test@example.com';
        $user->save();
        
        $this->assertNotNull($user->id);
    }
}
```

The `DatabaseTrait` provides:

- Runs migrations before all tests in the class
- Wraps each test in a database transaction
- Automatically rolls back changes after each test
- Ensures clean database state between tests

## Authentication Testing

Use `auth()->loginAs()` to simulate a logged-in user without requiring credentials.

```php
public function testAuthenticatedUserCanAccessDashboard()
{
    $user = new User(1);

    auth()->loginAs($user);
    
    $this->request('GET', '/dashboard');
    $this->assertResponseStatus(200);
}
```

Testing protected routes:

```php
public function testGuestCannotAccessProtectedRoute()
{
    $this->request('GET', '/admin/users');
    $this->assertResponseStatus(302);
    $this->assertRedirectUrl('/login');
}
```

Testing API authentication:

```php
public function testApiRequiresAuthentication()
{
    $this->requestJson('GET', '/api/users');
    $this->assertResponseStatus(401);
}

public function testApiWithValidToken()
{
    $this->withHeaders(['Authorization' => 'Bearer valid-token'])
         ->requestJson('GET', '/api/users');
    $this->assertResponseStatus(200);
}
```

## Method Chaining

All assertion methods return `$this`, allowing you to chain multiple assertions:

```php
$this->request('POST', '/login', [
        'email' => 'user@example.com',
        'password' => 'secret'
    ])
    ->assertResponseStatus(302)
    ->assertRedirectUrl('/dashboard')
    ->assertSessionHas('_logged_in', true)
    ->assertMailSent()
    ->assertMailSentTo('user@example.com');
```
