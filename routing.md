# Routing

Lightpack’s routing system lets you map incoming HTTP request URLs to appropriate controllers and actions with clarity and flexibility. This guide covers **all routing features** available to app developers—so you can build delightful, robust APIs and web apps.

---

## Quick Start

```php
// Basic GET route (defaults to 'index' action)
route()->get('/products', ProductController::class);

// Specify action
route()->get('/products', ProductController::class, 'list');

// Route with parameter
route()->get('/products/:id', ProductController::class, 'show');
```

> **Note:** All route methods (`get()`, `post()`, `put()`, etc.) default to the `'index'` action method if not specified.

---

## Route Files

Define routes in the `routes` folder:

```
routes/
  ├── web.php   // for web routes
  └── api.php   // for API routes
```

---

## Route Methods

Lightpack supports all HTTP verbs and flexible grouping:
- `route()->get()`
- `route()->post()`
- `route()->put()`
- `route()->patch()`
- `route()->delete()`
- `route()->options()`
- `route()->any()` (registers for all verbs)
- `route()->map()` (registers for multiple verbs)
- `route()->group()` (for prefix/filter/host grouping)

**Examples:**
```php
route()->post('/products', ProductController::class, 'store');
route()->map(['GET', 'POST'], '/contact', ContactController::class, 'handle');
route()->any('/ping', HealthController::class);
```

> **Important:** `map()` throws an `Exception` if you provide an unsupported HTTP verb (only GET, POST, PUT, PATCH, DELETE, OPTIONS are supported).

---

## Route Parameters

Define dynamic segments with `:param` syntax:
```php
route()->get('/users/:id', UserController::class, 'show');
```
- Parameters are extracted and passed to your controller action.
- Parameters are available as controller method arguments.
- Route parameters are also exposed via `request()->params('id')`.

**Optional parameters:**
```php
route()->get('/users/:id/photos/:photo?', UserController::class, 'photo');
```
- `?` makes the **last parameter** optional (will be `null` if not present).
- **Only the last parameter can be optional** — you cannot have optional parameters in the middle.

---

## Custom Regex & Placeholders

Use built-in placeholders or custom regex for flexible matching:
- `:any` → `.*` (matches anything)
- `:seg` → `[^/]+` (matches any segment except slash) — **default for all params**
- `:num` → `[0-9]+` (matches digits only)
- `:slug` → `[a-zA-Z0-9-]+` (matches URL-friendly slugs)
- `:alpha` → `[a-zA-Z]+` (matches letters only)
- `:alnum` → `[a-zA-Z0-9]+` (matches letters and digits)

**Using placeholders:**
```php
route()->get('/posts/:slug', PostController::class)->pattern(['slug' => ':slug']);
```

**Using custom regex:**
```php
route()->get('/users/:id', UserController::class)->pattern(['id' => '[0-9]{4}']);
```

> **Note:** If you don't specify a pattern, parameters default to `:seg` (matches any non-slash characters).

---

## Route Naming & URL Generation

Name routes for easy URL generation:
```php
route()->get('/products/:id', ProductController::class)->name('product.show');
```

Generate URLs using the `url()` helper:
```php
url()->route('product.show', ['id' => 42]); // /products/42
```

> **Important:** The second parameter must be an **associative array** with keys matching the route parameter names.

> **Important:** Route names must be unique. Registering a duplicate name throws an `Exception` with message: `"Duplicate route name: {name}"`.

---

## Route Filters

Filters are request interceptors that allow you to execute logic before or after a route is processed — perfect for authentication, validation, logging, or any cross-cutting concerns.

Attach filters to routes or groups:
```php
route()->get('/admin', AdminController::class)->filter('auth');
route()->post('/users', UserController::class)->filter(['auth', 'csrf']);
```

- Filters can be strings or arrays.
- Filters execute in the order they are defined.
- See [filters docs](filters.md) for details on creating and using filters.

---

## Route Groups

Group routes by prefix, filters, or host:
```php
// Prefix group
route()->group(['prefix' => '/api/v1'], function() {
    route()->get('/users', UserController::class);
    // Results in: /api/v1/users
});

// Filter group
route()->group(['filter' => ['auth']], function() {
    route()->post('/posts', PostController::class);
    // All routes inherit 'auth' filter
});

// Host-based group
route()->group(['host' => 'admin.example.com'], function() {
    route()->get('/dashboard', AdminController::class);
    // Only matches on admin.example.com
});
```

**Group behavior:**
- **Prefixes** concatenate (nested groups accumulate prefixes).
- **Filters** merge and deduplicate with `array_unique()`.
- **Host** is inherited if not set in nested group.
- Groups can be nested; options are merged.
- Prefix trimming handles slashes automatically.

---

## Multi-Verb Routes

Register a route for multiple HTTP verbs with `map()` or all verbs with `any()`:
```php
// GET and POST
route()->map(['GET', 'POST'], '/feedback', FeedbackController::class, 'submit');

// All verbs (GET, POST, PUT, PATCH, DELETE, OPTIONS)
route()->any('/status', StatusController::class);
```

> **Note:** `any()` registers the route for all supported HTTP verbs: GET, POST, PUT, PATCH, DELETE, OPTIONS.

---

## Subdomain / Host-Based Routing

You can restrict routes to specific hosts or subdomains:

**Group-level:**
```php
route()->group(['host' => 'api.example.com'], function() {
    route()->get('/users', UserController::class);
});
```

**Individual route:**
```php
route()->get('/admin', AdminController::class)->host('admin.example.com');
```

**Wildcard subdomains:**
```php
route()->group(['host' => ':subdomain.example.com'], function() {
    route()->get('/dashboard', DashboardController::class);
});
// Matches: tenant1.example.com, tenant2.example.com, etc.
// Subdomain available as parameter in controller method
```

The wildcard `:subdomain` extracts everything before `.example.com`:
- `tenant1.example.com` → `subdomain = 'tenant1'`
- `api.v2.example.com` → `subdomain = 'api.v2'` (multi-level subdomains work!)

Access the subdomain parameter in your controller:
```php
public function dashboard()
{
    $subdomain = request()->params('subdomain');
    // or as method parameter
}

public function dashboard($subdomain)
{
    // $subdomain is automatically injected
}
```

---

## Best Practices & Gotchas

- **Order matters:** Register more specific routes before generic ones.
- **Unique route names:** Duplicate names throw `Exception: "Duplicate route name: {name}"`.
- **Empty paths:** Empty route paths throw `Exception: "Empty route path"`.
- **Unsupported verbs:** `map()` throws `Exception: "Unsupported HTTP request method: {verb}"` for invalid verbs.
- **Filter accumulation:** Filters from groups and routes are merged and deduplicated with `array_unique()`.
- **Group nesting:** Prefixes concatenate, filters merge, host is inherited.
- **Optional params:** Only the **last** parameter can be optional using `:param?`; missing params are `null`.
- **Default pattern:** Parameters without explicit patterns default to `:seg` (matches `[^/]+`).
- **Host matching:** When a route has a host, matching checks `request()->host() . '/' . path`. Routes with hosts will NOT match if the request comes from a different host.
- **Wildcard subdomains:** The pattern `:subdomain.example.com` only matches hosts ending with `.example.com`. Other domains are rejected.
- **404s:** If no route matches, Lightpack throws a `RouteNotFoundException`.

---