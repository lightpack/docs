# Auth Filters

You might want to protect a **route** or a group of **routes** defined in your application. These routes should only execute for successfully authenticated users.

**Lightpack** ships with two filters: `WebFilter` and `ApiFilter`.

## WebFilter

Used as `auth:web` alias, this filter will check if the current session has been authenticated else will redirect to login page.

For example:

```php
route()->group(['filters' => 'auth:web'], function() {
    // protected routes list here
});
```

Use `auth:web` filter for **session-cookie** based routes authentication.

