# Configuration

Here is a brief explanation for **config/auth.php** configuration file.

## identifier

This key is the class name that represents a user identifier. This class implements `Lightpack\Auth\Identifier` interface and acts as a user data service provider.  You can also [implement your own custom](custom-auth) auth identifers.

## login.url

This key represents the **login** page route for session-cookie based authentication. 

## logout.url

This key represents the **logout** route for session-cookie based authentication. 

## login.redirect

This is the route where the client is **redirected** post successful login.

## logout.redirect

This is the page where the client is **redirected** post successful logout.

## fields.id

This key is the name for **user id** which by default, it is set to `id`.

## fields.username

This key is the form-field name for **username** input which by default, it is set to `email`.

## fields.password

This key is the form-field name for **password** input.

## fields.remember_token

This key is the name of the column in the users table that stores the remember-me token.

## remember_duration

This key sets the duration (in minutes) for how long the remember-me cookie remains valid. Default is **30 days** (`60 * 24 * 30`).

```php
'remember_duration' => 60 * 24 * 30, // 30 days
```

Examples:
- 7 days: `60 * 24 * 7`
- 90 days: `60 * 24 * 90`
- Never expire: omit this key (not recommended for security)

## flash_error

This key contains the default error message for **failed** login attempts.

---