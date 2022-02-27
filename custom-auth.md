# Extending Authentication

**Lightpack's** authentication system is made up of `Authenticators` and `Identifiers`. 

Understanding these two aspects will help you customize and implement your own authentication system as per your application needs. 

## Authenticators

Authenticators are classes that implement `Lightpack\Auth\Authenticator` interface. 

These classes are responsible for authenticating a request. You can create your own authenticator by implementing this interface. For example:

```php
<?php

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