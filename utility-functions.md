# Lightpack Utility Functions

Lightpack provides few handy utility functions to simplify common usage scenarios in your application.

## csrf_input()

**What it does:**
Outputs a hidden HTML input containing the current CSRF token (named `_token`).

**When to use:**
- Always include this in every `<form>` that performs a POST, PUT, PATCH, or DELETE request.
- Protects your application from Cross-Site Request Forgery attacks.

**Example:**
```php
<form method="POST">
  <?= csrf_input() ?>
  <!-- other fields -->
</form>
```

## csrf_token()

**What it does:**
Returns the current CSRF token as a string.

**When to use:**
- When you need to access the token value directly (e.g., for AJAX requests or custom headers).

**Example:**
```php
$token = csrf_token();
// Use $token in an AJAX request header
```

## spoof_input()

`spoof_input(string $method)`

**What it does:**
Outputs a hidden input to spoof HTTP methods (like PUT, PATCH, DELETE) in HTML forms.

**When to use:**
- When you want to send a method other than GET or POST in a form submission.
- HTML forms only support GET and POST natively; this lets you use RESTful verbs.

**Example:**
```php
<form method="POST" action="/resource/123">
  <?= csrf_input() ?>
  <?= spoof_input('DELETE') ?>
  <button type="submit">Delete</button>
</form>
```

## dd()

`dd(...$args)`

**What it does:**
“Dump and Die”—outputs the contents of variables (like `var_dump`) and stops the script.

**When to use:**
- For debugging: inspect variables, arrays, or objects at any point in your code.
- Use in development only; never leave in production code.

**Example:**
```php
dd($user, $posts);
```

## pp()

`pp(...$args)`

**What it does:**
“Pretty Print”—outputs variables using `print_r` and stops the script. This function is useful for debugging purposes, especially when working with arrays and objects.

**When to use:**
- For debugging: best for arrays and objects where structure matters. This provides a clear and readable representation of the data.
- Use in development only.

**Example:**
```php
pp($myArray);
```

```php
pp($var1, $var2, $var3)
```

## halt()

`halt(int $code, string $message = '', array $headers = [])`

**What it does:**
Immediately halts execution and throws an HTTP exception with the specified status code. If no message is provided, uses the standard HTTP status message (e.g., "Not Found" for 404).

**When to use:**
- When you need to stop request processing and return an HTTP error response.
- For access control (403 Forbidden), missing resources (404 Not Found), or server errors (500 Internal Server Error).
- As a clean way to handle error conditions in controllers, middleware, or route handlers.

**Example:**
```php
// Simple halt with standard message
halt(404);  // Throws "Not Found"
halt(403);  // Throws "Forbidden"

// With custom message
halt(404, 'User not found');
halt(403, 'You need admin access');

// With custom headers
halt(503, 'Service Unavailable', ['Retry-After' => '3600']);

// In conditional logic
if (!$user) {
    halt(404, 'User not found');
}

// One-liner
if (!$user) halt(404);
```

## once()

`once(callable $callback)`

**What it does:**
Executes a callback only once and caches the result. Subsequent calls with the same callback return the cached result without re-executing the callback.

**When to use:**
- Prevent duplicate expensive operations (database queries, API calls, file reads)
- Lazy loading with automatic caching
- Ensure idempotent operations within the same request

**Example:**
```php
// In a model - expensive query runs only once
class User
{
    public function permissions()
    {
        return once(function() {
            // This query only runs once per request
            return Permission::where('user_id', $this->id)->all();
        });
    }
}

$user->permissions(); // Executes query
$user->permissions(); // Returns cached result
$user->permissions(); // Returns cached result
```

```php
// Prevent duplicate API calls
$callback = function() {
    return $httpClient->get('https://api.example.com/data');
};

$data1 = once($callback); // Makes API call
$data2 = once($callback); // Returns cached response
$data3 = once($callback); // Returns cached response
```

```php
// Expensive computation
class Report
{
    private $calculateStats;
    
    public function __construct()
    {
        $this->calculateStats = function() {
            // Heavy computation
            return $this->processLargeDataset();
        };
    }
    
    public function getStats()
    {
        return once($this->calculateStats);
    }
}

$report = new Report();
$stats = $report->getStats(); // Computes
$stats = $report->getStats(); // Cached
```

**Important:**
- The callback must be the **same instance** to benefit from caching
- Different callback instances (even with identical code) are treated separately
- Cache persists only for the current request

## optional()

`optional($value, ?callable $callback = null)`

**What it does:**
Safely access properties and methods on a potentially null value without causing errors. Returns a null-safe object that allows chaining.

**When to use:**
- Accessing nested properties that might be null
- Preventing "Attempt to read property on null" errors
- Cleaner code without excessive null checks
- Working with optional relationships or API responses

**Example:**
```php
// Basic usage - no error even if $user is null
$name = optional($user)->name;
$email = optional($user)->profile->email;

// Without optional (verbose)
$name = $user !== null ? $user->name : null;
$email = $user !== null && $user->profile !== null ? $user->profile->email : null;

// With optional (clean)
$name = optional($user)->name;
$email = optional($user)->profile->email;
```

```php
// Real-world: fetching user from database
$user = User::find($id); // Might return null

// Safe access
$userName = optional($user)->name;
$userCity = optional($user)->address->city;
$userEmail = optional($user)->getEmail();

// All work without errors, even if $user is null
```

```php
// With callback transformation
$user = User::find($id);

$displayName = optional($user, function($u) {
    return strtoupper($u->firstName . ' ' . $u->lastName);
});

// If $user is null, $displayName is the null object
// If $user exists, $displayName is the transformed string
```

```php
// In views - prevent errors from missing data
<h1><?= optional($post->author)->name ?></h1>
<p><?= optional($order->customer->address)->city ?></p>

// In controllers
public function show($id)
{
    $user = User::find($id);
    
    return response()->json([
        'name' => optional($user)->name,
        'email' => optional($user)->profile->email,
        'city' => optional($user)->address->city,
    ]);
}
```

```php
// Chaining with method calls
$user = User::find($id);

// No error even if getProfile() returns null
$email = optional($user->getProfile())->email;
$phone = optional($user->getProfile())->getContactInfo()->phone;
```

**Important:**
- The null object converts to empty string when cast: `(string)optional(null)->property === ''`
- For actual null checks, use `is_null()` or strict comparison
- Works with properties, methods, and deep chaining
- Callback is only executed if value is not null

---