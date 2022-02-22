# Authentication

> Session-cookie or token based authentication made easy.

**Lightpack** supports **user/api** authentication in a very friendly manner. It exposes a couple of authentication methods using `auth()` function.

```php
auth()->id();
auth()->user();
auth()->login();
auth()->logout();
auth()->token();
auth()->recall();
auth()->attempt();
```

Also the configuration for authentication is present in `config/auth.php` file. In most cases, you can use these methods without any configurations needed.

Let's understand what each of these methods are and when to use them.

