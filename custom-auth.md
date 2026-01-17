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
- If your authentication method differs from email/password (e.g., username/password, phone/OTP, LDAP).
- If you need to fetch users from a different source (API, external database, etc.)
- If you need different database column names than the framework conventions.

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
        // Register custom authenticator
        auth()->extend('jwt', JwtAuthenticator::class);

        // Use it
        $user = auth()->attempt();
        
        if (!$user) {
            return redirect()->to('login');
        }
        
        return redirect()->intendedRoute('dashboard');
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

Configure your custom identifier in `config/auth.php`:

```php
return [
    'auth' => [
        'drivers' => [
            'default' => [
                // ...
            ],
            'custom' => [
                'model' => App\Models\User::class,
                'identifier' => CustomIdentifier::class,
                'remember_duration' => 60 * 24 * 30,
            ],
        ],
        'routes' => [
            'login' => 'login',
            'home' => 'dashboard',
        ],
    ],
];
```

To use your custom driver, call `setDriver()` before authentication:

```php
namespace App\Controllers;

class LoginController
{
    public function authenticate()
    {
        $user = auth('custom')->attempt();
        
        if (!$user) {
            session()->flash('error', 'Invalid credentials.');
            return redirect()->to('login');
        }
        
        return redirect()->intendedRoute('dashboard');
    }
}
```

## Quick Summary

| Component | Purpose | How to Customize |
|-----------|---------|------------------|
| **Authenticator** | Verifies requests (JWT, OAuth, etc.) | Extend `AbstractAuthenticator`, register with `extend()` |
| **Identifier** | Fetches users from data source | Implement `Identifier` interface, add to config |
| **Driver** | Combines model + identifier + config | Add new driver in `config/auth.php` |

**Default Setup:**
- Authenticators: `FormAuthenticator`, `CookieAuthenticator`, `BearerAuthenticator`
- Identifier: `EmailPasswordIdentifier`
- Conventions: `email`, `password`, `remember_token`, `last_login_at` columns

---