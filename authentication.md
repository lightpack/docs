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

## Logging In

### Web Based Login
To login a user via **session-cookie** mechanism, call the `login()` method on **auth** object.

```php
auth()->login();
```

Behind the scenes, this method performs following actions:

* Check if `email/password` credentials match in **users** table.
* On **success**, start a new session and redirect to **post-login** url.
* On **failure**, redirect to the login page again with **error** message in session.

### API Based Login

To authenticate a user without maintaining **session-cookie**, use `attempt()` method.

```php
auth()->attempt();
```

This is useful when authenticating user via API requests. Behind the scenes, this method performs following actions:

* Check if `email/password` credentials match in **users** table.
* On **success**, returns an API **token** in JSON response.
* On **failure**, returns a **failure** response as JSON.

## Logging Out

To logout, simply call `logout()` method.

```php
auth()->logout();
```

## Authenticated User

To access currently authenticated user's **id**, call `id()` method.

```php
auth()->id();
```

To access currently authenticated **user**, call `user()` method.

```php
auth()->user();
```

To access currently authenticated user's **api token** , call `token()` method.

```php
auth()->token();
```

This method will return `null` when logged-in via **session-cookie** mechanism. So use this
method only when logged-in with `attempt()` method for API based login requests.

