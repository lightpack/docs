# Extending Authentication

**Lightpack's** authentication system is made up of `Authenticators` and `Identifiers`. 

Understanding these two aspects will help you customize and implement your own authentication system as per your application needs. 

## Authenticators

Authenticators are classes that extend `Lightpack\Auth\AbstractAuthenticator` class. 

These classes are responsible for authenticating a request. You can create your own authenticator by extending this class. For example:

```php
<?php

namespace App\Security;

use Lightpack\Auth\Identity;
use Lightpack\Auth\AbstractAuthenticator;

class CustomAuthenticator extends AbstractAuthenticator
{
    public function verify(): ?Identity
    {
        // custom auth logic goes here
    }
}
```

The `verify()` method should return an instantce of **\Lightpack\Auth\Identity**. 

## Identifiers

Identifiers are classes that implement `Lightpack\Auth\Identifier` interface.

These classes are responsible for **fetching** users and they act as user repository or user data service providers. You can created your own identifier by implementing this interface. For example:

```php
<?php

namespace App\Security;

use Lightpack\Auth\Identity;
use Lightpack\Auth\Identifier;

class CustomIdentifier implements Identifier
{
    public function findById($id): ?Identity
    {
        // custom logic to fetch user by id
    }

    public function findByAuthToken(string $token): ?Identity
    {
        // fetch user with matching api_token
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

## Configuration

Now the final step remains is to configure your custom authentication provider. Add a new key in `config/auth.php` file
with a value that identifies your authentication privider.

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

Once configured, call the `extend()` method on `auth()` in your login controller or wherever you want to use this custom authentication. For example:

```php
<?php 

namespace App\Controllers;

class LoginController
{
    public function authenticate()
    {
        // Use custom authentication
        auth()->extend('custom');

        // Try to validate login
        auth()->login();
    }
}
```