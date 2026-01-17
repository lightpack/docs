# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** provides a simple to use authentication system for authenticating web and api requests. The `auth()` helper exposes authentication methods:

```php
// Check authentication status
auth()->isLoggedIn();  // Returns true if user is logged in
auth()->isGuest();     // Returns true if user is not logged in

// Get authenticated user
auth()->id();          // Returns logged in user ID
auth()->user();        // Returns logged in user object

// Authenticate users
auth()->attempt();     // Verify credentials and persist session
auth()->loginAs($user); // Login as specific user (testing/impersonation)
auth()->viaToken();    // Authenticate via Bearer token (API)
auth()->recall();      // Auto-login via remember-me cookie

// Logout
auth()->logout();      // Clear session and cookies
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
| `attempt()` | `Identity\|null` | ✅ Yes | ✅ Yes (if remember me) | Web form login |
| `loginAs($user)` | `Auth` | ✅ Yes | ❌ No | Testing/Admin impersonation |
| `viaToken()` | `Identity\|null` | ❌ No | ❌ No | Bearer token authentication |
| `recall()` | `Identity\|null` | ✅ Yes (if valid) | ❌ No | Auto-login from cookie |

### Web Form Login

To authenticate a user via **session-cookie** mechanism, use `attempt()` in your controller:

```php
public function login(LoginRequest $request)
{
    $user = auth()->attempt();

    if (!$user) {
        session()->flash('error', 'Invalid account credentials.');
        return redirect()->to('login');
    }

    // Redirect to intended URL, or 'dashboard' route if none
    return redirect()->intendedRoute('dashboard');
}
```

#### Login Request Validation

The `LoginRequest` class extends `FormRequest` to validate login credentials before attempting authentication:

```php
class LoginRequest extends FormRequest
{
    protected function rules()
    {
        $this->validator
            ->field('email')
            ->required()
            ->email();

        $this->validator
            ->field('password')
            ->required();
    }
}
```

**How it works:**

1. When the `login()` method is called, the `LoginRequest` is automatically validated
2. If validation **fails**: The user is redirected back with error messages
3. If validation **passes**: The controller method executes

This ensures that:
- The `email` field is present and contains a valid email address
- The `password` field is present

You can customize validation rules based on your requirements. See [Validation](validation.md) for more details on available rules.

#### Authentication Flow

Behind the scenes, `attempt()` performs the following:

1. Retrieves `email` and `password` from request input
2. Verifies credentials against the **users** table
3. On **success**:
   - Returns the authenticated user (`Identity` object)
   - Creates a session with `_logged_in` and `_auth_id`
   - Updates `last_login_at` timestamp
   - Sets remember-me cookie (if `remember` input is checked)
4. On **failure**: Returns `null`

**Your controller** is responsible for:
- Handling redirects
- Setting flash messages
- Deciding where to send the user

### Remember Me

To enable "remember me" functionality, include a checkbox in your login form:

```html
<input type="checkbox" name="remember">
<label>Remember me</label>
```

When checked, `attempt()` will set a long-lived cookie that allows automatic login on future visits.

### API Login

For API authentication, use `attempt()` to verify credentials, then create an access token:

```php
public function login()
{
    $identity = auth()->attempt();

    if (!$identity) {
        return response()->json(['error' => 'Invalid credentials'], 401);
    }

    // Create API token
    $token = $identity->createToken('api-token');
    
    return response()->json([
        'token' => $token->plainTextToken,
        'user' => $identity,
    ]);
}
```

**Note:** While `attempt()` creates a session, API clients typically ignore cookies and use the returned token for subsequent requests.

### Bearer Token Authentication

To authenticate API requests with a **Bearer** token, use `viaToken()`:

```php
public function getProfile()
{
    $user = auth()->viaToken();

    if (!$user) {
        return response()->json(['error' => 'Unauthorized'], 401);
    }

    return response()->json(['user' => $user]);
}
```

This method:
1. Extracts the Bearer token from the `Authorization` header
2. Verifies the token against the `access_tokens` table
3. Returns the authenticated user or `null`
4. Updates `last_used_at` timestamp on the token
5. Updates `last_login_at` on the user

### Direct Login (Testing/Impersonation)

To log in as a specific user ID without credentials:

```php
$user = new User(1);
auth()->loginAs($user);
```

This is useful for:
- **Testing**: Quickly authenticate as different users
- **Admin impersonation**: Allow admins to view the app as a specific user

This method:
- Creates a session immediately
- Updates `last_login_at` timestamp
- Does NOT set remember-me cookie
- Returns the `Auth` instance for chaining

## Logging Out

To logout a user, call `logout()` in your controller:

```php
public function logout()
{
    auth()->logout();
    return redirect()->route('login');
}
```

The `logout()` method:
- Clears the user identity
- Deletes the remember-me cookie
- Destroys the session
- Returns `void` (your controller handles the redirect)

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

## Auto-Login (Recall)

The `recall()` method attempts to automatically log in a user from a remember-me cookie:

```php
public function dashboard()
{
    // Try to auto-login from remember-me cookie
    auth()->recall();

    if (auth()->isGuest()) {
        return redirect()->route('login');
    }

    return view('dashboard');
}
```

Behind the scenes, `recall()`:
1. Checks if user is already logged in via session
   - If **yes**: Returns the user (no action needed)
2. If **no**: Checks for a valid `remember_token` cookie
   - If **valid**: Logs the user in and returns the user
   - If **invalid/missing**: Returns `null`

**Note:** The `AuthFilter` automatically calls `recall()`, so you typically don't need to call it manually if the filter applied on route.

### Cookie Duration

By default, remember-me cookies last for **30 days**. Configure the duration in `config/auth.php`:

```php
'remember_duration' => 60 * 24 * 30, // 30 days in minutes
```

See [Configuration](auth-configuration.md#remember_duration) for more details.

---