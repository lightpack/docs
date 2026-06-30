# Lightpack SocialAuth: Complete Developer Guide

> **Lightpack SocialAuth** provides seamless OAuth authentication for your users via Google, GitHub, LinkedIn, and more. Designed with Lightpack’s philosophy of clarity, explicitness, and extensibility, it supports both web and stateless API flows, and is easy to extend for custom providers.

## Supported Providers
Out of the box:
- Google
- GitHub
- LinkedIn


## Environment Variables

Add the following variables to your `.env` file based on the providers you use:

```env
APP_URL=https://your-app.com

# Google
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# GitHub
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret

# LinkedIn
LINKEDIN_CLIENT_ID=your-linkedin-client-id
LINKEDIN_CLIENT_SECRET=your-linkedin-client-secret
```

The `redirect_uri` for each provider is auto-generated from `APP_URL`:
- `https://your-app.com/auth/google/callback`
- `https://your-app.com/auth/github/callback`
- `https://your-app.com/auth/linkedin/callback`

Make sure to register these exact redirect URIs in your OAuth app settings.

## Configuration

Please run following command to create `config/social.php` configuration file.

```cli
php console create:config --support=social
```

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
use Lightpack\SocialAuth\Controllers\SocialAuthController;

// Web
route()->get('/auth/{provider}/redirect', [SocialAuthController::class, 'redirect']);
route()->get('/auth/{provider}/callback', [SocialAuthController::class, 'callback']);

// API (stateless)
route()->get('/api/auth/{provider}/redirect', [SocialAuthController::class, 'redirect']);
route()->get('/api/auth/{provider}/callback', [SocialAuthController::class, 'callback']);
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

## Multi-Tenancy

The SocialAuth module is fully multi-tenancy aware. When used in a multi-tenant application, social accounts are automatically scoped by `tenant_id`.

### How It Works

- The `social_accounts` table includes a `tenant_id` column (defaults to `0` for non-tenant apps).
- The unique constraint is scoped to `(tenant_id, provider, provider_id)`, allowing the same social account to be linked across different tenants.
- During redirect, the current `tenant_id` is stored in session (web) or encoded in the OAuth `state` (API).
- During callback, the `tenant_id` is restored from session or state, and `TenantContext` is set.
- The `findOrCreateUser()` method scopes social account lookups by `tenant_id` and assigns `tenant_id` to new social accounts and users.

### Preparing Your User Model

For tenant-aware social authentication, extend `TenantModel`:

```php
use Lightpack\Database\Lucid\TenantModel;

class User extends TenantModel
{
    // ...
}
```

### What You Don't Need To Do

- No manual `tenant_id` assignment in routes or controllers.
- No extra query scoping needed; the framework handles isolation automatically.

### Backward Compatibility

Non-tenant apps continue to work seamlessly. `tenant_id` defaults to `0`, ensuring no breaking changes.

## Adding a Custom Provider

Create a new provider class implementing `SocialAuthInterface`:
   
```php
namespace App\SocialAuth\Providers;

use Lightpack\SocialAuth\SocialAuthInterface;

class MyProvider implements SocialAuthInterface
{
    public function getAuthUrl(array $params = []): string
    {
        // Build and return the OAuth authorization URL
    }

    public function getUser(string $code): array
    {
        // Exchange code for access token, fetch user profile
        // Return: ['id' => ..., 'name' => ..., 'email' => ..., 'avatar' => ...]
    }

    public function stateless(): self
    {
        // Set stateless flag for API flows, return $this
    }
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