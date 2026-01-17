# Auth Filters

Protect routes or groups of routes so they only execute for authenticated users. **Lightpack** provides two built-in filters: `AuthFilter` for protected routes and `GuestFilter` for guest-only routes.

## AuthFilter

The `AuthFilter` protects routes by ensuring users are authenticated. It supports both **web** (session-based) and **API** (token-based) authentication.

### Web Authentication

For session-based routes, use `auth:web`:

```php
route()->group(['filters' => 'auth:web'], function() {
    // protected routes list here
});
```

**How it works:**

1. Checks if user is logged in via session
2. If **not logged in**:
   - Stores the current URL as "intended" (for GET requests)
   - Attempts auto-login via remember-me cookie (`recall()`)
   - If recall fails, redirects to login page
3. If **logged in**: Allows the request to proceed

### API Authentication

For API routes, use `auth:api`:

```php
route()->group(['filters' => 'auth:api'], function() {
    // protected routes list here
});
```

**How it works:**

1. Extracts Bearer token from `Authorization` header
2. Validates token via `auth()->viaToken()`
3. If **invalid**: Returns `401 Unauthorized` JSON response
4. If **valid**: Allows the request to proceed

### Configuring Redirect Routes

The `AuthFilter` uses named routes from your `config/auth.php` to determine where to redirect unauthenticated users:

```php
'routes' => [
    'login' => 'login',  // Redirect here when not authenticated
],
```

Make sure you have a corresponding named route:

```php
route()->get('/login', AuthController::class, 'showLogin')->name('login');
```

See [Configuration](auth-configuration.md#routes-configuration) for more details.

## GuestFilter

The `GuestFilter` ensures only **unauthenticated** users can access certain routes. This is useful for login, registration, and password reset pages.

```php
route()->group(['filters' => 'guest'], function() {
    // routes only for guests (not logged-in users)
});
```

**How it works:**

1. Checks if user is logged in
2. If **logged in**: Redirects to the home page
3. If **not logged in**: Allows the request to proceed

### Configuring Authenticated Route

The `GuestFilter` uses the `routes.authenticated` config to determine where to redirect authenticated users:

```php
'routes' => [
    'authenticated' => 'dashboard',  // Redirect here when already authenticated
],
```

Make sure you have a corresponding named route:

```php
route()->get('/dashboard', DashboardController::class, 'index')->name('dashboard');
```

## Per-Route Filters

You can apply filters to individual routes:

```php
$router->get('/admin', [AdminController::class, 'index'])
    ->filter('auth:web');

$router->get('/login', [AuthController::class, 'showLogin'])
    ->filter('guest');
```
## Remember Me Behavior

The `AuthFilter` automatically attempts to log in users via their remember-me cookie. This means:

1. User visits a protected route
2. No active session found
3. Valid remember-me cookie exists
4. User is automatically logged in
5. Request proceeds normally

This provides a seamless experience for users who checked "Remember me" during login.

---