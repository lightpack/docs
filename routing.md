# Routing

Lightpack’s routing system lets you map incoming HTTP request URLs to appropriate controllers and actions with clarity and flexibility. This guide covers **all routing features** available to app developers—so you can build delightful, robust APIs and web apps.

---

## Quick Start

```php
// Basic GET route
route()->get('/products', ProductController::class); // maps to index()

// Specify action
route()->get('/products', ProductController::class, 'list');

// Route with parameter
route()->get('/products/:id', ProductController::class, 'show');
```

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

---

## Route Parameters

Define dynamic segments with `:param` syntax:
```php
route()->get('/users/:id', UserController::class, 'show');
```
- Parameters are passed as arguments to your controller action.

**Optional parameters:**
```php
route()->get('/users/:id/photos/:photo?', UserController::class, 'photo');
```
- `?` makes the last parameter optional (will be `null` if not present).

---

## Custom Regex & Placeholders

Use built-in placeholders or custom regex for flexible matching:
- `:any` → `.*`
- `:num` → `[0-9]+`
- `:seg` → `[^/]+`
- `:slug` → `[a-zA-Z0-9-]+`
- `:alpha` → `[a-zA-Z]+`
- `:alnum` → `[a-zA-Z0-9]+`

**Example:**
```php
route()->get('/posts/:slug', PostController::class)->pattern(['slug' => ':slug']);
```
**Custom regex:**
```php
route()->get('/users/:id', UserController::class)->pattern(['id' => '([0-9]{4})']);
```

---

## Route Naming & URL Generation

Name routes for easy URL generation:
```php
route()->get('/products/:id', ProductController::class)->name('product.show');
```
Generate URLs using the `url()` helper:
```php
url()->route('product.show', 42); // /products/42
```

---

## Route Filters

Attach filters (middleware) to routes or groups:
```php
route()->get('/admin', AdminController::class)->filter('auth');
route()->post('/users', UserController::class)->filter(['auth', 'csrf']);
```
- Filters can be strings or arrays; they accumulate and are unique per route.
- See [filters docs](filters.md) for details.

---

## Route Groups

Group routes by prefix, filters, or host:
```php
// Prefix group
route()->group(['prefix' => '/api/v1'], function() {
    route()->get('/users', UserController::class);
});

// Filter group
route()->group(['filter' => ['auth']], function() {
    route()->post('/posts', PostController::class);
});

// Host-based group
route()->group(['host' => 'admin.example.com'], function() {
    route()->get('/dashboard', AdminController::class);
});
```
- Groups can be nested; options are merged (prefixes/filters/hosts accumulate).

---

## Multi-Verb Routes

Register a route for multiple HTTP verbs with `map()` or all verbs with `any()`:
```php
// GET and POST
route()->map(['GET', 'POST'], '/feedback', FeedbackController::class, 'submit');

// All verbs
route()->any('/status', StatusController::class);
```

---

## Best Practices & Gotchas

- **Order matters:** Register more specific routes before generic ones.
- **Unique route names:** Avoid duplicate names for reliable URL generation.
- **Filter accumulation:** Filters from groups and routes are merged and deduplicated.
- **Group nesting:** Prefixes and filters accumulate in nested groups.
- **Optional params:** Use `:param?` for optional last segment; missing param are `null`.
- **Host-based routes:** Use `host` option for subdomain or domain-specific routes.
- **404s:** If no route matches, Lightpack throws a `RouteNotFoundException`.

---