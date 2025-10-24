# API Access Tokens

API access tokens are secure, random strings issued to users or applications that allow them to authenticate and access protected resources over HTTP APIs. They are commonly used for stateless authentication in RESTful APIs, mobile apps, and third-party integrations. Each token can be associated with specific permissions ("abilities" or "scopes") and may have an expiration time, enabling fine-grained access control and easy revocation without relying on session state.

---

## Authenticate API

Authenticate API requests using a bearer token provided in the `Authorization` header. This is the standard approach for stateless authentication in APIs.

**How it works:**
- The client sends an HTTP header: `Authorization: Bearer <token>`.
- The server extracts the token and verifies it against the `access_tokens` table.
- If valid and not expired, the associated user is authenticated for the request.

```php
// Authenticate the request using a bearer token
$user = auth()->viaToken();

if ($user) {
    // The token is valid
}
```

## Issue Access Tokens

A user can have one or more access tokens allowing them to authenticate future requests. Tokens can have specific abilities (scopes) and expiration dates.

**How it works:**
- Call `createToken()` on the user object, specifying a name, an array of abilities (permissions), and an optional expiration date.
- The method creates a hashed token in the database and returns a temporary `plainTextToken` for the client to store securely.
- The wildcard `'*'` grants all abilities.

```php
$user = auth()->user(); // Get the currently authenticated user
```

Following issues a never expiring API access token named **site-manager** with all the abilities/permissions.

```php
$token = $user->createToken('site-manager');
```

Following issues a token that expires in 30 days with limited abilities.

```php
$token = $user->createToken(
    'site-manager',                
    ['read:post', 'write:post'], 
    moment()->travel('+30 days')
);
```

**You should return the plain text token to the client:**

```php
return ['token' => $token->plainTextToken];
```

> **Note:** The `plainTextToken` property is only available immediately after creation. The client should store it securely as it cannot be retrieved again.

---

## Check Token Abilities

You can authorize specific actions based on the abilities (scopes) assigned to the access token. This enables fine-grained access control for API clients.

**How it works:**
- Each token has an array of abilities (e.g., `['read:post', 'write:post']`).
- When handling a request, check if the authenticated user's token grants the required ability.

```php
if ($user->tokenCan('write:post')) {
    // User can write posts
}
```

```php
if ($user->tokenCannot('write:post')) {
    // User cannot write posts
}
```

## Delete Tokens

Tokens can be deleted/revoked individually or in bulk.

```php
// Delete all tokens for the user
$user->deleteTokens();        

// Delete a specific token by its plaintext value
$user->deleteTokens($plainTextToken); 

// Delete current request access token 
$user->deleteCurrentRequestToken();

// Delete a token by ID
$user->deleteTokensById(23);

// Delete multiple tokens by ID
$user->deleteTokensById([23, 24, 24]);
```

- Deleting tokens immediately invalidates them for future authentication.
- Use this to implement logout-from-all-devices or token revocation endpoints.

## Token Tracking

Access tokens automatically track usage information for monitoring and debugging:

```php
$token = AccessToken::find($id);

echo $token->last_used_at;  // Timestamp of last authentication
```

**Use cases:**
- **Detect stale tokens**: Find tokens not used in 90+ days
- **Security monitoring**: Track when/how tokens are used
- **Debugging**: "When was this token last active?"

```php
// Find inactive tokens
$staleTokens = AccessToken::query()
    ->where('last_used_at', '<', date('Y-m-d H:i:s', strtotime('-90 days')))
    ->get();
```

## Best Practices

### 1. Use Namespaced Abilities

Instead of generic abilities like `read` or `write`, use a `resource:action` format:

```php
$token = $user->createToken('api', [
    'posts:read',
    'posts:write',
    'users:read',
    'comments:delete',
]);
```

**Benefits:**
- Clear what resource and action are allowed
- Easy to understand permissions
- Industry standard (OAuth scopes use this pattern)

### 2. Set Expiration Dates

Tokens should expire to limit security exposure:

```php
// Recommended: 30-90 days
$token = $user->createToken(
    'mobile-app',
    ['*'],
    date('Y-m-d H:i:s', strtotime('+30 days'))
);
```

### 3. Monitor Token Usage

Regularly check `last_used_at` to detect and revoke stale tokens:

```php
// Revoke tokens not used in 6 months
$staleTokens = AccessToken::query()
    ->where('user_id', $userId)
    ->where('last_used_at', '<', date('Y-m-d H:i:s', strtotime('-6 months')))
    ->get();

foreach ($staleTokens as $token) {
    $token->delete();
}
```

### 4. Limit Token Count

Consider limiting how many tokens a user can have:

```php
$tokenCount = AccessToken::query()
    ->where('user_id', '=', $user->id)
    ->count();

if ($tokenCount >= 10) {
    return response()->json(['error' => 'Maximum token limit reached'], 400);
}
```

### 5. Revoke on Logout

Delete tokens when user logs out or changes password:

```php
// Logout from all devices
$user->deleteTokens();

// Or delete current token only
$user->deleteCurrentRequestToken();
```
  
---