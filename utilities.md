# Lightpack Utilities

Lightpack provides few handy utility functions to simplify common usage scenarios in your application.

## csrf_input()

**What it does:**
Outputs a hidden HTML input containing the current CSRF token (named `_token`).

**When to use:**
- Always include this in every `<form>` that performs a POST, PUT, PATCH, or DELETE request.
- Protects your application from Cross-Site Request Forgery attacks.

**Example:**
```php
<form method="POST">
  <?= csrf_input() ?>
  <!-- other fields -->
</form>
```

## csrf_token()

**What it does:**
Returns the current CSRF token as a string.

**When to use:**
- When you need to access the token value directly (e.g., for AJAX requests or custom headers).

**Example:**
```php
$token = csrf_token();
// Use $token in an AJAX request header
```

## spoof_input()

`spoof_input(string $method)`

**What it does:**
Outputs a hidden input to spoof HTTP methods (like PUT, PATCH, DELETE) in HTML forms.

**When to use:**
- When you want to send a method other than GET or POST in a form submission.
- HTML forms only support GET and POST natively; this lets you use RESTful verbs.

**Example:**
```php
<form method="POST" action="/resource/123">
  <?= csrf_input() ?>
  <?= spoof_input('DELETE') ?>
  <button type="submit">Delete</button>
</form>
```

## old()

`old(string $key, $default = '', bool $escape = true)`

**What it does:**
Returns the previously submitted value for a form field, or a default if not present.

**When to use:**
- To repopulate form fields after a validation error, so users don’t lose their input.
- Especially useful in large forms or when validation fails.

**Example:**
```php
<input name="email" value="<?= old('email') ?>">
```
If the user submitted the form and it failed validation, their email input will be preserved.

## error()

`error(string $key)`

**What it does:**
Returns the validation error message for a specific field, if any.

**When to use:**
- To show users what went wrong with their input after a failed form submission.
- Place near each form field for clear feedback.

**Example:**
```php
<input name="email" value="<?= old('email') ?>">
<span class="error"><?= error('email') ?></span>
```
If validation fails for `email`, the error message appears next to the field.

## _e()

`_e(string $str)`

**What it does:**
Escapes HTML special characters to prevent XSS attacks.

**When to use:**
- Always escape any user-generated or dynamic content before outputting it in HTML.
- Use for comments, user names, or any data you don’t fully control.

**Example:**
```php
<p><?= _e($comment->body) ?></p>
```

## dd()

`dd(...$args)`

**What it does:**
“Dump and Die”—outputs the contents of variables (like `var_dump`) and stops the script.

**When to use:**
- For debugging: inspect variables, arrays, or objects at any point in your code.
- Use in development only; never leave in production code.

**Example:**
```php
dd($user, $posts);
```

## pp()

`pp(...$args)`

**What it does:**
“Pretty Print”—outputs variables using `print_r` and stops the script. This function is useful for debugging purposes, especially when working with arrays and objects.

**When to use:**
- For debugging: best for arrays and objects where structure matters. This provides a clear and readable representation of the data.
- Use in development only.

**Example:**
```php
pp($myArray);
```

```php
pp($var1, $var2, $var3)
```