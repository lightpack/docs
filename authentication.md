# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** supports authentication in a very friendly manner. It exposes a couple of authentication methods using `auth()` function.

```php
// Returns logged in user id
auth()->id();

// Returns logged in user 
auth()->user();

// Returns true if user is logged in
auth()->isLoggedIn();

// Returns true if a user is not logged in
auth()->isGuest();

// Attempt login and start session
auth()->login();

// Login as a specific user without credentials (useful for testing/impersonation)
auth()->loginAs($user);

// Attempt login once without starting session
auth()->attempt();

// Logout the user 
auth()->logout();

// Attempts to automatically login based on remember me cookie
auth()->recall();

// Verify incoming request bearer token
auth()->viaToken();

// Redirect URL post successful login
auth()->redirectLogin();

// Redirect URL post logout
auth()->redirectLogout();

// Redirect to login URL
auth()->redirectLoginUrl();
```

## Installation

Before utilizing **auth** feature support, you need to migrate and publish configuration files as documented below.

### Migration

Migrate required schema in order to use **authentication** features.

Create schema migration file:

```cli
php console create:migration --support=users
```

Run migration:

```cli
php console migrate:up
```

### Configuration

Please run following command to create **config/auth.php** file.

```php
php console create:config --support=auth
```

## Logging In

### Login Methods Comparison

Different login methods serve different purposes. Choose the right one for your use case:

| Method | Returns | Sets Session | Sets Cookie | Use Case |
|--------|---------|--------------|-------------|----------|
| `login()` | `Redirect` | ✅ Yes | ✅ Yes (if remember me) | Traditional web forms |
| `attempt()` | `Identity\|null` | ❌ No | ❌ No | API/AJAX requests |
| `loginAs($user)` | `Auth` | ✅ Yes | ❌ No | Testing/Admin impersonation |
| `viaToken()` | `Identity\|null` | ❌ No | ❌ No | Bearer token authentication |

### Web Based Login
To login a user via **session-cookie** mechanism, call the `login()` method on **auth** object.

```php
auth()->login();
```

Behind the scenes, this method performs following actions:

* Check if `email/password` credentials match in **users** table.
* On **success**, start a new session and redirect to **post-login** url.
* On **failure**, redirect to the login page again with **error** message in session.

### API Based Login

To authenticate a user without maintaining **session-cookie**, use `attempt()` method.

```php
$user = auth()->attempt();

if ($user) {
    // Authentication successful - create token
    $token = $user->createToken('api');
    return response()->json(['token' => $token->plainTextToken]);
}

return response()->json(['error' => 'Invalid credentials'], 401);
```

This is useful when authenticating user via API requests based on **username/password** credentials. Behind the scenes, this method performs following actions:

* Check if `email/password` credentials match in **users** table.
* On **success**, returns the authenticated **user** object (or `null` on failure).
* Does NOT start a session or set cookies.
* You must manually create an API token and return it to the client.

#### Bearer Token

To authenticate an API request containing **Bearer** token in authorization header, use:

```php
auth()->viaToken();
```

## Logging Out

To logout, simply call `logout()` method.

```php
auth()->logout();
```

## Authenticated User

To access currently authenticated user's **id**, call `id()` method.

```php
auth()->id();
```

To access currently authenticated **user**, call `user()` method.

```php
auth()->user();
```

> **Note:** The `user()` method works for both session-based and token-based authentication.

## Remember Me

If the user checks the **remember** functionality while logging in, you can use `recall()` method.

```php
auth()->recall();
```

Behind the scenes, this method performs the following actions:

* Checks if the user is already logged in via session.
* If **yes**, redirects to the **post-login** URL.
* If **no**, checks if a `remember_me` cookie is present and valid.
* If the **cookie is valid**, the user is automatically logged in (a new session is started) and redirected to the **post-login** URL.
* If the **cookie is missing or invalid**, the user is redirected to the **login** page.

### Cookie Duration

By default, remember me cookies last for **30 days**. You can configure the duration in `config/auth.php` using the `remember_duration` key. See [Configuration](auth-configuration.md#remember_duration) for details.

---