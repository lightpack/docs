# Extending Authentication

**Lightpack's** authentication system is made up of two key building blocks: `Authenticators` and `Identifiers`.

Understanding how these work together will help you confidently customize authentication for your application.

## Concepts

**Authenticator**
- Handles the mechanics of authentication (e.g., parsing a JWT, validating a password, checking a cookie)
- Extracts identifying information (e.g., user ID) from the request

**Identifier**
- Responsible for fetching the user (Identity) from a data source, given some identifying info (like user ID, email, etc.)
- Is agnostic to how the identifying info was obtained

**How do they work together?**
- Authenticator verifies the request and extracts the identifier (e.g., user ID from JWT)
- Identifier loads the user from the database (or other source)
- This separation allows you to mix and match authentication strategies and user sources

**When do you need a custom Identifier?**
- Only if your user-fetching logic is different from the default (e.g., you want to look up by email instead of ID, or fetch from an API)
- Most of the time, the default identifier is sufficient—even for JWT authentication

**Why this separation?**
- Encourages single responsibility and testability
- Makes it easy to add new authentication methods without rewriting user lookup logic
- Enables advanced scenarios (multi-tenancy, external user stores, etc.)

Below we document in detail how to define your own custom authenticators and identifiers as required.

## Authenticators

Authenticators are classes that extend `Lightpack\Auth\AbstractAuthenticator` class. 

These classes are responsible for authenticating a request. You can create your own authenticator by extending this class. For example:

```php
class JwtAuthenticator extends AbstractAuthenticator
{
    public function verify(): ?Identity
    {
        // custom auth logic goes here
    }
}
```

The `verify()` method should return an instance of **\Lightpack\Auth\Identity** (or `null` if authentication fails). 

To use your custom authenticator, you should register it using the `extend()` method. For example, in your login controller:

```php
class LoginController
{
    public function authenticate()
    {
        // specify authenticator to use
        auth()->extend('jwt', JwtAuthenticator::class);

        return auth()->login();
    }
}
```

## Identifiers

Identifiers are classes that implement `Lightpack\Auth\Identifier` interface.

These classes are responsible for **fetching** users and they act as user repository or user data service providers. You can create your own identifier by implementing this interface. For example:

```php
class CustomIdentifier implements Identifier
{
    public function findById($id): ?Identity
    {
        // custom logic to fetch user by id
    }

    public function findByRememberToken($id, string $token): ?Identity
    {
        // fetch user with matching remember_me token
    }

    public function findByCredentials(array $credentials): ?Identity
    {
        // fetch user by username/password credentials
    }

    public function updateLogin($id, array $fields)
    {
        // update login fields 
    }
}
```

### Configuration

Now the final step is to configure your custom identifier. Add a new key in your `config/auth.php` file with a value that identifies your authentication provider:

```php
<?php

return [
    'auth' => [
        'default' => [
            // ...
        ],
        'custom' => [
            'identifier' => CustomIdentifier::class,
        ],
    ]
];
```

To use your custom provider in a controller or any part of your app, simply call `auth('custom')` to switch to your provider for that call chain:

```php
<?php

namespace App\Controllers;

class LoginController
{
    public function authenticate()
    {
        return auth('custom')->login();
    }
}
```

## Quick Summary Table

| Task                    | How to do it                                           |
|-------------------------|--------------------------------------------------------|
| Register authenticator  | `auth()->extend('jwt', JwtAuthenticator::class);`      |
| Use custom provider     | `auth('custom')->login();`                             |
| Register identifier     | Add to `config/auth.php` under `'custom'`              |

---