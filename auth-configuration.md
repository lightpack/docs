# Configuration

Here is a brief explanation for **config/auth.php** configuration file.

## Configuration Structure

The auth configuration file contains two main sections:

```php
return [
    'auth' => [
        'drivers' => [
            'default' => [
                'model' => App\Models\User::class,
                'identifier' => Lightpack\Auth\Identifiers\EmailPasswordIdentifier::class,
                'remember_duration' => 60 * 24 * 30,
            ],
        ],

        'routes' => [
            'guest' => 'login',
            'authenticated' => 'dashboard',
        ],
    ],
];
```

## Driver Configuration

### model

The fully qualified class name of your user model. This model should implement the `Lightpack\Auth\Identity` interface.

```php
'model' => App\Models\User::class,
```

### identifier

The class name that handles user authentication logic. This class implements `Lightpack\Auth\Identifier` interface and is responsible for:

- Finding users by ID
- Finding users by credentials (email/password)
- Finding users by remember token
- Updating login timestamps

Lightpack ships with `EmailPasswordIdentifier` which authenticates users via email and password. You can [implement your own custom identifier](custom-auth) for different authentication methods (username/password, phone/OTP, etc.).

```php
'identifier' => Lightpack\Auth\Identifiers\EmailPasswordIdentifier::class,
```

### remember_duration

Sets the duration (in minutes) for how long the remember-me cookie remains valid. Default is **30 days** (`60 * 24 * 30`).

```php
'remember_duration' => 60 * 24 * 30, // 30 days
```

**Examples:**
- 7 days: `60 * 24 * 7`
- 90 days: `60 * 24 * 90`
- 1 year: `60 * 24 * 365`

> **Note:** This is optional. If omitted, it defaults to 30 days.

## Routes Configuration

The `routes` section configures which named routes the auth filters should redirect to. This allows you to customize redirect behavior without modifying framework code.

### routes.guest

The **named route** for your login page. The `AuthFilter` redirects unauthenticated users here.

```php
'guest' => 'login', // Route name, not URL
```

Make sure you have a corresponding route defined:

```php
route()->get('/login', [AuthController::class, 'showLogin'])->name('login');
```

### routes.authenticated

The **named route** where authenticated users are redirected. The `GuestFilter` uses this when logged-in users try to access guest-only pages (like login or registration).

```php
'authenticated' => 'dashboard', // Route name, not URL
```

Make sure you have a corresponding route defined:

```php
route()->get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

---