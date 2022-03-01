# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** supports authentication in a very friendly manner. It exposes a couple of authentication methods using `auth()` function.

```php
// Returns logged in user id
auth()->id();

// Returns logged in user 
auth()->user();

// Returns true if user is logged in
auth()->isLoggedIn();

// Attempt login and start session
auth()->login();

// Attempt login once without starting session
auth()->attempt();

// Logout the user 
auth()->logout();

// Returns auth token when logged in via api
auth()->token();

// Attempts to automatically login based on remember me cookie
auth()->recall();

// Verify incoming request bearer token
auth()->viaToken();

// Redirect URL post successful login
auth()->redirectLogin();

// Redirect URL post logout
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

This is useful when authenticating user via API requests based on **username/password** credentials. Behind the scenes, this method performs following actions:

* Check if `email/password` credentials match in **users** table.
* On **success**, returns an API **token** in JSON response.
* On **failure**, returns a **failure** response as JSON.

#### Bearer Token

To authenticate an API request containing **Bearer** token in authorization header, use:

```php
auth()->viaToken();
```

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

## Remember Me

If the user checks the **remember** functionality while logging in, you can use `recall()` method.

```php
auth()->recall();
```

Behind the scenes, this method will perform following actions.

* Check if **session** has logged in.
* On **success**, redirect to **post-login** url.
* On **failure**, check if `remember_me` cookie is set and valid.
* If **cookie** is valid, redirect to **post-login** url.
* Else, redirect to **login** page.