# Sessions

<p class="tip">Lightpack provides the <code>session()</code> function to work with sessions in a consistent, secure, and flexible way. The session system supports multiple storage drivers, dot notation for nested data, CSRF and agent validation, and more.</p>

## Quick Start

To use sessions, just call the global <code>session()</code> helper:

```php
session()->set('user_id', 42);
$userId = session()->get('user_id');
```

## Supported Methods

```php
session()->set()
session()->get()
session()->has()
session()->delete()
session()->flash()
session()->regenerate()
session()->destroy()
session()->token()
session()->hasInvalidToken()
session()->hasInvalidAgent()
session()->verifyAgent()
session()->setUserAgent()
```

---

## Features & Usage

### Setting Session Data

Set a value (including arrays/objects) for a key:

```php
session()->set('key', $value);
```

Supports <strong>dot notation</strong> for nested data:

```php
session()->set('user.profile.name', 'Alice');
```

### Getting Session Data

Get a value by key:

```php
session()->get('key');
```

Get a nested value:

```php
session()->get('user.profile.name');
```

Provide a default if the key isn’t found:

```php
session()->get('key', 'default');
```

Get all session data:

```php
session()->get();
```

### Checking Existence

Check if a key (or nested key) exists:

```php
session()->has('key');
session()->has('user.profile.name');
```

### Deleting Session Data

Delete a key (or nested key):

```php
session()->delete('key');
session()->delete('user.profile.name');
```

### Flash Data

Flash data persists only for the next request (great for messages):

```php
session()->flash('notice', 'Profile updated!'); // Set flash data
$notice = session()->flash('notice'); // Get and remove flash data
```

### Regenerating Session ID

Regenerate the session ID (for security after login, etc):

```php
session()->regenerate();
```
- <strong>Note:</strong> Also deletes the CSRF token.

### Destroying the Session

Completely destroy the session and all its data:

```php
session()->destroy();
```

---

## Security Features

### CSRF Token

Generate or retrieve a CSRF token:

```php
$token = session()->token();
```

Check if the CSRF token is invalid (e.g., after form submission):

```php
if(session()->hasInvalidToken()) {
    // Block the request!
}
```

### User Agent Validation

Store and verify the user agent string to help prevent session hijacking:

```php
// Manually set agent
session()->setUserAgent('AppleWebKit/KHTML'); 

if(session()->hasInvalidAgent()) {
    // Block the request!
}
```

## Advanced Features

### Dot Notation for Nested Data

You can set, get, check, or delete deeply nested session data using dot notation:

```php
session()->set('cart.items.0.product_id', 123);
$productId = session()->get('cart.items.0.product_id');
session()->delete('cart.items.0.product_id');
session()->has('cart.items.0.product_id');
```

### Driver System

Lightpack sessions support multiple drivers, each with different storage backends:

- <strong>DefaultDriver:</strong> Uses PHP’s native <code>$_SESSION</code> (file-based).
- <strong>ArrayDriver:</strong> Stores data in-memory (great for tests).
- <strong>CacheDriver:</strong> Stores session in a cache backend.
- <strong>RedisDriver:</strong> Uses Redis for scalable, distributed sessions.
- <strong>EncryptedDriver:</strong> Encrypts session values at rest.

You can configure the driver in your app’s config.

## Edge Cases & Notes

- <strong>Session keys can be any string.</strong> Use dot notation for nested arrays.
- <strong>Flash data</strong> is removed after it is read.
- <strong>Regenerating</strong> deletes the CSRF token for safety.
- <strong>If you use a custom driver</strong>, it must implement the <code>DriverInterface</code>.
- <strong>Session ID and cookie settings</strong> (name, lifetime, security flags) are configurable.
- <strong>EncryptedDriver</strong> requires a <code>Crypto</code> instance and will serialize/deserialize values automatically.
- <strong>ArrayDriver</strong> is not persistent and should only be used for testing.
- <strong>CacheDriver</strong> and <strong>RedisDriver</strong> handle session IDs and cookies internally, and support TTL (expiry).

---

## Configuration

Session settings (driver, name, lifetime, security, etc.) are controlled in your app’s config file, typically <code>config/session.php</code>.

---