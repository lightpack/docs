# Auth Filters

You might want to protect a **route** or a group of **routes** defined in your application. These routes should only execute for successfully authenticated users.

**Lightpack** ships with two filters: `WebFilter` and `ApiFilter`.

## WebFilter

Used as `auth:web` alias, this filter will check if the current session has been authenticated else will redirect to login page.

For example:

```php
route()->group(['filters' => ['auth:web']], function() {
    // protected routes list here
});
```

Use `auth:web` filter for **session-cookie** based routes authentication.

## ApiFilter

Used as `auth:api` alias, this filter should be used to protect API routes. This filter will look for **Bearer** token in authorization header and attempt to match it in **users** table `api_token` column. 

If authentication fails, it will abort the request and return a `401` unauthorized JSON response. 

For example:

```php
route()->group(['filters' => 'auth:api'], function() {
    // protected routes list here
});
```