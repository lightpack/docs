# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** supports **user/api** authentication in a very friendly manner. It exposes a couple of authentication methods using `auth()` function.

```php
// get logged in user id
auth()->id();

// Gets logged in user 
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
```

Also the configuration for authentication is present in `config/auth.php` file. In most cases, you can use these methods without any configurations needed.