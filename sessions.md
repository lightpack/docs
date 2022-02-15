# Sessions

<p class="tip">Lightpack provides <code>session()</code> function to work with sessions.</p>

You can access following functions to deal with sessions:

```php
session()->set()
session()->get()
session()->has()
session()->delete()
session()->regenerate()
session()->flash()
session()->token()
session()->hasInvalidToken()
session()->hasInvalidAgent()
session()->destroy()
```

## Set

To set a data in session, use <code>set()</code> method.

```php
session()->set('key', $value);
```

## Get

To get a data in session, use <code>get()</code> method.

```php
session()->get('key');
```

You can provide a default data to be returned if session key not found.

```php
session()->get('key', $default);
```

To get all session data, simply call <code>get()</code> method without specifying any key.

```php
session()->get();
```

## Has

To check if a session key exists, use <code>has()</code> method. It returns a boolean TRUE or FALSE.

```php
session()->has('key');
```

## Delete

To delete a session data, call <code>delete()</code> method by passing it the key value.

```php
session()->delete('key');
```

## Flash

To set a session data which should be available only till the next request, use <code>flash()</code>
method.

```php
session()->flash('key', $value);
```

To access the set flash data, simply call the same <code>flash()</code> method by passing it the key.

```php
session()->flash('key');
```

## CSRF Token

CSRF tokens are one of the security measures to verify session validity. To get 
a CSRF token, call <code>token()</code> method. This will generate a CSRF token
and set it in the session data by key <code>csrf-token</code>.

```php
<form method="post">
    <input 
        type="hidden" 
        name="csrf-token" 
        value="<?= session()->token() ?>"
    >
</form>
```

Now when the form is submitted, you can check the validity of CSRF token using <code>hasInvalidToken()</code>
method. This returns boolean TRUE if the token has been tampered otherwise false.

```php
if(session()->hasInvalidToken()) {
    // Ask the user to retry form submission, right?
}
```

## Verify Agent

Verifying session agent is also one of the security measures to verify session validity. To check
if the current session user agent is valid, use <code>hasInvalidAgent</code> method.

```php
if(session()->hasInvalidAgent()) {
    // May be block the request, right?
}
```

## Regenerate

To regenerate the current session ID, call <code>regenerate()</code> method.

```php
session()->regenerate();
```

## Destroy

To completely destroy current session, call <code>destroy()</code> method.

```php
session()->destroy();
```