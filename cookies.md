# Cookies

<p class="tip">Lightpack provides <code>app('cookie')</code> function to work with cookies.</p>

Note that all the cookies created by Lightpack is <code>encrypted</code> for better security. You can access
following functions to deal with cookies:

```php
app('cookie')->set()
app('cookie')->get()
app('cookie')->has()
app('cookie')->forever()
app('cookie')->delete()
```

## Set

This sets a cookie that expires at the end of current session:

```php
app('cookie')->set('key', 'value');
```

You can pass the number of seconds as expiry time for the cookie:

```php
app('cookie')->set('key', 'value', 5*60);
```

You can pass an array as fourth parameter to provide cookie options <code>path</code>, <code>domain</code>, <code>secure</code>, <code>http_only</code>.

```php
app('cookie')->set('key', 'value', 5*60, ['path' => $path, 'domain' => $domain]);
```

## Get

To access all the set cookies together, simply call <code>get()</code> method.

```php
app('cookie')->get();
```

To access a specific cookie, call it's <code>get()</code> method passing it name 
of the cookie. It returns <code>null</code> if cookie not found.

```php
app('cookie')->get('key');
```

You can specify a default value for a specific cookie if not found.

```php
app('cookie')->get('key', 'default');
```            

## Has

To check if a cookie is set, call it's <code>has()</code> method. It returns a boolean TRUE or FALSE.

```php
app('cookie')->has('key');
```

## Forever

Sometimes you may require to set a non-expiring cookie. One example is while implementing "Remember Me"
functionality when user logins. For that simply call <code>forever()</code> method. This automatically 
sets an expiry time long enough to set a non-expiring cookie. 


```php
app('cookie')->forever('key', 'value');
```

You can also pass an array as third parameter to pass other cookie options.

```php
app('cookie')->set('key', 'value', ['path' => $path, 'domain' => $domain]);
```

## Delete

To delete a specific cookie, call <code>delete()</code> method.

```php
app('cookie')->delete();
```