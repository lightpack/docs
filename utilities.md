# Utilities

`Lightpack` provides few handy utility functions.

## url()

This function exposes methods to generate URLs for your application.

To generate a URL, use `to()` method. It takes any number of string texts as arguments and concats them to generate relative URL to application basepath. 

```php
url()->to('products', 'edit', 23); // /products/edit/23
```

To generates relative URL with support for query parameters, pass an array as the last argument.

```php
url()->to('products', 'list', ['sort' => 'asc', 'status' => 'active']);
```

That will produce `/products/list?sort=asc&status=active`

To generate relative path URL for your application assets residing in `/public/assets` folder:

```php
url()->asset('css/styles.css'); // /assets/css/styles.css
url()->asset('img/favicon.png'); // /assets/img/favicon.png
url()->asset('js/scripts.js'); // /assets/js/scripts.js
```

## redirect()

Use this function for HTTP redirect relative to your applications domain. You can optionally pass HTTP redirect status code as second argument which defaults to `302`.

For example, to redirect to `/products/list`:

```php
redirect()->to('products/list');
```

## csrf_input()

Returns an HTML input element for CSRF token. Especially helpful when working with forms.

```php
<?php echo csrf_input(); ?>
```

This will generate an hidden input element with name `csrf_token` and a random CSRF token as value. For example:

```php
<input type="hidden" name="csrf_token" value="642c75336eece81c">
```

## str()

This function exposes methods to manipulate strings.


Use method `slugify()` to convert an ASCII text to URL friendly slug. It will replace any non-word character, non-digit character, or a non-dash '-' character with empty. Also it will replace any number of space character with a single dash '-'.

```php
str()->slugify('Hello World'); // hello-world
```

Use method `humanize()` This function converts a slug text to friendly human readable text. It replaces dashes and underscores with whitespace character. Then capitalizes the first character.

```php
str()->humanize('hello-world'); // Hello World
```

## get_env()

This function return an environment variable set in one of `$_ENV` or `$_SERVER` variable. 

It returns `null` in case he environment variable is not found.

```php
get_env('DB_HOST');
```

You can pass a default value as second argument to return.

```php
get_env('DB_HOST', 'localhost');
```

## set_env()

This function sets an environment variable.

```php
set_env('APP_VERSION', 'v1.0');
```

## _e()

This function is an HTML characters to entities converter.Use this function to escape HTML output to protect against XSS attacks.

```php
_e('<script>alert("dangerous")</script>');
```

## dd()

Use this function as a quick alternate to `var_dump()`. It will pretty dump the variables and then exit the PHP execution.

You can pass any number of variables to this function.

```php
var_dump($var1, $var2, ..., $varN);
```

## pp()

Use this function as a quick alternate to `print_r()`. It will pretty print the variables and then exit the PHP execution.

You can pass any number of variables to this function.

```php
print_r($var1, $var2, ..., $varN);
```