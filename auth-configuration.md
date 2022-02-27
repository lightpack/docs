# Configuration

**Lightpack** ships with default configuration for authentication in `config/auth.php` file.

You can check this file and find the `default` key which is configured for default authentication provided
by the framework itself.

```php
'default' => [
    'identifier'      => DefaultIdentifier::class,
    'login.url'       => '/login',
    'logout.url'      => '/logout',
    'login.redirect'  => '/',
    'logout.redirect' => '/login',
    'fields.username' => 'email',
    'fields.password' => 'password',
    'flash_error'     => 'Invalid account credentials.',
],
```

Here is a brief explanation for those configurations:

## identifier

This key is the class name that represents a user identifier. This class implements `Lightpack\Auth\Identifier` interface and contains following methods:

```php
public function findByAuthToken(string $token);
public function findByRememberToken($id, string $token);
public function findByCredentials(array $credentials);
public function updateLogin($id, array $fields);
```

You can create your own custom user data service providers to implement your own authentication mechanism.

## login.url

This key represents the **login** page route for session-cookie based authentication. 

## logout.url

This key represents the **logout** route for session-cookie based authentication. 

## login.redirect

This is the route where the client is **redirected** post successful login.

## logout.redirect

This is the page where the client is **redirected** post successful logout.

## fields.username

This key is the form-field name for **username** input which by default, it is set to `email`.

## fields.password

This key is the form-field name for **password** input.

## flash_error

This key contains the default error message for **failed** login attempts.