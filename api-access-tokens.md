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
  
---