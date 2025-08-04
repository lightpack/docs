# Lightpack SocialAuth: Complete Developer Guide

> **Lightpack SocialAuth** provides seamless OAuth authentication for your users via Google, GitHub, LinkedIn, and more. Designed with Lightpack’s philosophy of clarity, explicitness, and extensibility, it supports both web and stateless API flows, and is easy to extend for custom providers.

## Supported Providers
Out of the box:
- Google
- GitHub
- LinkedIn

Check the social auth related configuration in `config/social.php` file.

> In order to use the Google social auth feature, install the Google API Client package `composer require google/apiclient`

## Migration

Create schema migration file:

```cli
php console create:migration --support=social
```

Run migration:

```cli
php console migrate:up
```

## Usage Patterns

### 1. Routes
Define routes in your `routes/web.php` or `routes/api.php`:
```php
// Web
$route->get('/auth/{provider}/redirect', SocialAuthController::class, 'redirect');
$route->get('/auth/{provider}/callback', SocialAuthController::class, 'callback');

// API (stateless)
$route->get('/api/auth/{provider}/redirect', [SocialAuthController::class, 'redirect']);
$route->get('/api/auth/{provider}/callback', [SocialAuthController::class, 'callback']);
```

### 2. Controller Usage
The `SocialAuthController` is already shipped with this feature. It handles both web and API flows with these two supported methods:
- **redirect($provider):**
    - Web: Stores provider in session, redirects to provider auth URL.
    - API: Returns JSON with `auth_url`.
- **callback($provider):**
    - Web: Handles callback, logs in user, redirects to dashboard.
    - API: Returns access token and user info.

### Example: Web Flow
1. User clicks “Sign in with Google” → `/auth/google/redirect`
2. Redirects to Google OAuth
3. On success, Google redirects back to `/auth/google/callback`
4. User is logged in and redirected to dashboard

### Example: API Flow
1. Mobile app requests `/api/auth/google/redirect` (gets `auth_url`)
2. User authenticates in browser, Google redirects to `/api/auth/google/callback?code=...&state=...`
3. API returns JSON: `{ access_token, token_type, user }`

---

## Adding a Custom Provider

Create a new provider class implementing `SocialAuth`:
   
```php
namespace App\SocialAuth\Providers;

use Lightpack\SocialAuth\SocialAuth;

class MyProvider implements SocialAuth {
    // Implement the interface methods
}
```

Add config:
   
```php
'providers' => [
    'myprovider' => [
        'provider' => App\SocialAuth\Providers\MyProvider::class,
        // ... 
    ],
]
```

**And you’re done!**

---