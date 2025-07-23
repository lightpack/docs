# Extending Authentication

**Lightpack's** authentication system is made up of `Authenticators` and `Identifiers`. 

Understanding these two aspects will help you customize and implement your own authentication system as per your application needs. 

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