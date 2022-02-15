# Cookies

<p class="tip">Lightpack provides <code>cookie()</code> function to work with cookies.</p>

Note that all the cookies created by Lightpack is <code>encrypted</code> for better security. You can access
following functions to deal with cookies:

```php
cookie()->set()
cookie()->get()
cookie()->has()
cookie()->forever()
cookie()->delete()
```

## Set

This sets a cookie that expires at the end of current session:

```php
cookie()->set('key', 'value');
```

You can pass the number of seconds as expiry time for the cookie:

```php
cookie()->set('key', 'value', 5*60);
```

You can pass an array as fourth parameter to provide cookie options <code>path</code>, <code>domain</code>, <code>secure</code>, <code>http_only</code>.

```php
cookie()->set('key', 'value', 5*60, ['path' => $path, 'domain' => $domain]);
```

## Get

To access all the set cookies together, simply call <code>get()</code> method.

```php
cookie()->get();
```

To access a specific cookie, call it's <code>get()</code> method passing it name 
of the cookie. It returns <code>null</code> if cookie not found.

```php
cookie()->get('key');
```

You can specify a default value for a specific cookie if not found.

```php
cookie()->get('key', 'default');
```            

## Has

To check if a cookie is set, call it's <code>has()</code> method. It returns a boolean TRUE or FALSE.

```php
cookie()->has('key');
```

## Forever

Sometimes you may require to set a non-expiring cookie. One example is while implementing "Remember Me"
functionality when user logins. For that simply call <code>forever()</code> method. This automatically 
sets an expiry time long enough to set a non-expiring cookie. 


```php
cookie()->forever('key', 'value');
```

You can also pass an array as third parameter to pass other cookie options.

```php
cookie()->set('key', 'value', ['path' => $path, 'domain' => $domain]);
```

## Delete

To delete a specific cookie, call <code>delete()</code> method.

```php
cookie()->delete();
```