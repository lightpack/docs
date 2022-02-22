# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** supports **user/api** authentication in a very friendly manner. It exposes a couple of authentication methods using `auth()` function.

```php
// Returns logged in user id
auth()->id();

// Returns logged in user 
auth()->user();

// Attempt login and start session
auth()->login();

// Delete current session and logout the user 
auth()->logout();

// Returns auth token when logged in via api
auth()->token();

// Attempts to automatically login based on remember me cookie
auth()->recall();

// Attempt login once without starting session
auth()->attempt();

// Verify incoming request bearer token in database
auth()->viaToken();

// Redirect URL after successful login
auth()->redirectLogin();

// Redirect URL after logout
auth()->redirectLogout();

// Redirect to login URL
auth()->redirectLoginUrl();
```

Also the configuration for authentication is present in `config/auth.php` file. In most cases, you can use these methods without any configurations needed.