# Filters

Filters in Lightpack are reusable hooks that run before or after a controller action. They are typically used for authentication, authorization, CSRF protection, rate limiting, CORS, and other request/response concerns.

## How Filters Work

- **Before filters** can halt the request and return a response before the controller action runs.
- **After filters** can modify the response after the controller action runs.
- Filters can be attached to individual routes or route groups.

## Usage

### Attaching Filters to Routes

```php
// Single filter
route()->get('/dashboard', DashboardController::class)->filter('auth');

// Multiple filters
route()->post('/posts', PostController::class)->filter(['csrf', 'auth']);
```

### Grouping Filters

```php
route()->group(['filter' => ['csrf', 'auth']], function() {
    route()->get('/dashboard', DashboardController::class);
});
```

### Halting or Modifying Requests

- To halt a request early (e.g., unauthorized), return a `Response` from the filter’s `before()` method.
- To modify the response, return a new `Response` from the filter’s `after()` method.

---

## Defining Custom Filters

```cli
php console create:filter TrimFilter
```

A filter is a class implementing the `Lightpack\Filters\IFilter` interface:

```php
class TrimFilter implements IFilter
{
    public function before(Request $request, array $params = [])
    {
        // Logic before action
    }

    public function after(Request $request, Response $response, array $params = []): Response
    {
        // Logic after action
        return $response;
    }
}
```

**Register your filter alias in `config/filters.php`:**

```php
return [
    'trim' => App\Filters\TrimFilter::class,
];
```

---

## Built-in Filters

Lightpack provides many pre-defined filters that you can declare on your route definitions. Below are the built-in filters provided by Lightpack, their aliases, and what they do:

### 1. `auth`
**Purpose:** Restricts access to authenticated users (web or API).
- **Web:** Redirects guests to login; stores intended URL for redirect after login.
- **API:** Returns 401 JSON if token is missing or invalid.
- **Params:** `['web']` (default) or `['api']`.

### 2. `guest`
**Purpose:** Restricts access to guests only.
- Redirects logged-in users to a default location (usually dashboard).

### 3. `csrf`
**Purpose:** Protects against CSRF attacks.
- Checks for a valid CSRF token on POST, PUT, PATCH, DELETE.
- Throws exceptions for missing or invalid tokens.
- **Bypasses** in `APP_ENV=testing`.

### 4. `cors`
**Purpose:** Handles Cross-Origin Resource Sharing.
- Responds to OPTIONS requests with CORS headers.
- Adds CORS headers to all responses.
- Uses headers from `config('cors.headers')`.

### 5. `limit`
**Purpose:** API rate limiting.
- Limits requests per user or IP to a configurable max per time window.
- Throws 429 with rate limit headers if exceeded.
- **Params:** `[maxRequests, minutes]`. Defaults from `config('limit.default')`.

### 6. `signed`
**Purpose:** Ensures URL signatures are valid.
- Returns 403 (JSON or error view) if the request signature is invalid.

### 7. `verifyemail`
**Purpose:** Restricts access to users with verified email addresses.
- Redirects or returns 403 JSON if user’s email is not verified.

### 8. `mfa`
**Purpose:** Enforces Multi-Factor Authentication.
- If MFA is enforced or user has enabled MFA, triggers MFA flow and redirects to verification screen.
- Skips if already passed in session.

---

## Filter Parameters

You can pass parameters to filters via the route definition:

```php
route()->get('/api/data', ApiController::class)->filter(['limit:100,5']);
```
- For `limit`, this would allow 100 requests per 5 minutes.

---

## Best Practices

- **Order matters:** Filters run in the order they’re listed.
- **Return a Response:** To halt further processing, return a `Response` from `before()`.
- **Chainable:** Filters from groups and routes are merged and deduplicated.
- **Testing:** CSRF filter is disabled in test environment.

---

## Summary Table

| Alias         | Description                            | Typical Use/Params         |
|---------------|----------------------------------------|---------------------------|
| `auth`        | Require authentication (web/API)       | `auth:web`, `auth:api`    |
| `guest`       | Require guest (not logged in)          |                           |
| `csrf`        | CSRF protection on state-changing verbs|                           |
| `cors`        | Add CORS headers, handle preflight     |                           |
| `limit`       | Rate limiting                         | `limit:60,1`              |
| `signed`      | Require valid signed URL               |                           |
| `verifyemail` | Require verified email                 |                           |
| `mfa`         | Enforce multi-factor authentication    |                           |

---