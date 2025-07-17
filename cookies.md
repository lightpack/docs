# Cookies

<p class="tip">Lightpack provides the <code>cookie()</code> function to work with cookies.</p>

**Note:** All cookies created by Lightpack are <strong>signed</strong> using HMAC for tamper detection. The cookie value is visible to the client, but any modification will be detected and rejected.

You can access the following functions:

```php
cookie()->set()
cookie()->get()
cookie()->has()
cookie()->forever()
cookie()->delete()
```

## Set

Set a cookie (expires at the end of the session):

```php
cookie()->set('key', 'value');
```

Set a cookie with custom expiry (in seconds):

```php
cookie()->set('key', 'value', 5*60);
```

Set a cookie with options (<code>path</code>, <code>domain</code>, <code>secure</code>, <code>http_only</code>, <code>same_site</code>):

```php
cookie()->set('key', 'value', 5*60, [
    'path' => $path,
    'domain' => $domain,
    'secure' => true,
    'http_only' => true,
    'same_site' => 'lax', // or 'strict', 'none'
]);
```

## Get

Get all cookies:

```php
cookie()->get();
```

Get a specific cookie (returns <code>null</code> if not found or tampered):

```php
cookie()->get('key');
```

## Has

Check if a cookie is set:

```php
cookie()->has('key');
```

## Forever

Set a cookie that lasts for years (useful for “Remember Me”):

```php
cookie()->forever('key', 'value');
```

You can also pass options as the third parameter:

```php
cookie()->forever('key', 'value', ['path' => $path, 'same_site' => 'strict']);
```

## Delete

Delete a specific cookie:

```php
cookie()->delete('key');
```

---

<strong>Security Note:</strong>  
If a cookie is modified by the client, Lightpack will detect the tampering and return <code>null</code> for that cookie.

---