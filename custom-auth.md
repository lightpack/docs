# Extending Authentication

**Lightpack's** authentication system is made up of `Authenticators` and `Identifiers`. 

Understanding these two aspects will help you customize and implement your own authentication system as per your application needs. 

## Authenticators

Authenticators are classes that implement `Lightpack\Auth\Authenticator` interface. 

These classes are responsible for authenticating a request. You can create your own authenticator by implementing this interface. For example:

```php
<?php

namespace App\Security;

use Lightpack\Auth\Result;
use Lightpack\Auth\Identifier;
use Lightpack\Auth\Authenticator;

class CustomAuthenticator implements Authenticator
{
    public function verify(Identifier $identifier, array $config)
    {
        // custom auth logic goes here
    }
}
```

The `verify()` method should return **null** on failed authentication. If authentication succeeds, it should return authenticated user details.

## Identifiers

Identifiers are classes that implement `Lightpack\Auth\Identifier` interface.

These classes are responsible for **fetching** users and they act as user repository or user data service providers. You can created your own identifier by implementing this interface. For example:

```php
<?php

namespace App\Security;

class CustomIdentifier implements Identifier
{
    public function findByAuthToken(string $token)
    {
        // fetch user with matching api_token
    }

    public function findByRememberToken($id, string $token)
    {
        // fetch user with matching remember_me token
    }

    public function findByCredentials(array $credentials)
    {
        // fetch user by username/password credentials
    }

    public function updateLogin($id, array $fields)
    {
        // update login fields 
    }
}
```