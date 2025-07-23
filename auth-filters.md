# Auth Filters

You might want to protect a **route** or a group of **routes** defined in your application. These routes should only execute for successfully authenticated users.

**Lightpack** ships with `auth` filter for **web** and **api** routes. You can use this filter to authenticate per route definition or on a group of routes.

## Web Filter

Used as `auth:web` alias, this filter will check if the current session has been authenticated else will redirect to login page.

For example:

```php
route()->group(['filters' => ['auth:web']], function() {
    // protected routes list here
});
```

Use `auth:web` filter for **session-cookie** based routes authentication.

## Api Filter

Used as `auth:api` alias, this filter should be used to protect API routes. This filter will look for **Bearer** token in authorization header and attempt to authenticate the validity of the token.

If authentication fails, it will abort the request and return a `401` unauthorized JSON response. 

For example:

```php
route()->group(['filters' => 'auth:api'], function() {
    // protected routes list here
});
```

## Guest Filter

The `guest` filter ensures that only unauthenticated users (guests) can access certain routes. If an authenticated user tries to access a route protected by this filter, they will be redirected to the login landing page (or wherever your `auth()->redirectLogin()` is configured to go).

Use this filter for routes like login, registration, or password reset—pages that should not be accessible to already logged-in users.

For example:

```php
route()->group(['filters' => 'guest'], function() {
    // routes only for guests (not logged-in users)
});
```

---