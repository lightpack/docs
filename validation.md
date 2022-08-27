# Data Validation

When accepting data from a source such as form input, you may want to validate data
itself to ensure you get the data as expected. `Lightpack` gives you a simple approach
to validate data for most of the use cases. 

<p class="tip">While you are free to use any data validation library of your choice, Lightpack provides <code>Lightpack\Validator\Validator</code> class for data validation which you might find handy enough
for your data validation needs.</p>

## Quick example

First create an instance of `Lightpack\Validator\Validator` passing it the data source as an array
of `key, value` pairs. For example, in case of a form submission via `POST` method, the data source 
can be `$_POST`.

```php
<?php

use Lightpack\Validator\Validator;

$validator = new Validator($_POST);
```

Now set validation rules per field

```php
$validator->setRules([
    'name' => 'required|alpha',
    'email' => 'required|email',
    'password' => 'required|min:8',
]);
```

Finally run validation

```php
$validator->run();
```

Check if data validation failed.

```php
$validator->hasErrors(); // boolean
```

Get all validation errors as an array

```php
$validator->getErrors();
```

Now that you have a quick start with data validation in `Lightpack`, let us go through docs in detail.

## Setting Rules

You can specify rules per field either individually using `setRule()` or all together at once using 
`setRules()` method.

### Setting rules individually

To set rules for an individual field, call `setRule()` method:

```php
$validator->setRule('email', 'required|email');
$validator->setRule('password', 'required|min:8');
```

You can also chain calls to `setRule()` method:

```php
$validator
    ->setRule('email', 'required|email');
    ->setRule('password', 'required|min:8');
```

### Setting rules together

To set rules for multiple fields together, call `setRules()` method:

```php
$validator->setRules([
    'email' => 'required|email',
    'password' => 'required|min:8',
]);
```

## Running Validation

Once you are done specifying rules, you first need to call `run()` method to process
data validation.

```php
$validator->run();
```

This will process validation rules for all fields and populate appropriate error messages
in case a field data violates a rule. You can customize these error messages which has been 
documented below.

## Validation Errors

You can check if validation has failed by calling `hasErrors()` method. This method returns
a boolean `TRUE` if validation failed, otherwise `FALSE`.

```php
if($validator->hasErrors()) {
    // ...
}
```

**NOTE**: You can chain together the calls to `run()` and `hasErrors()` methods.

```php
if($validator->run()->hasErrors()) {
    // ...
}
```

### Getting validation errors

To get all error messages together as an array, call `getErrors()` method.

```php
$validator->getErrors(); 
```

To get error message for an individual field, call `getError()` method. It returns the error message
string text or an empty string if the field has no validation error.

```php
$validator->getError('email');
```

### Custom error messages

By default the validation will create error messages for fields
which will be sufficient enough in most cases. But if you want to customize an error
message for a field, you can do that too.

To customze an error message for a field, pass an array of rules and custom message.

```php
$validator->setRules([
    'email' => [
        'rules' => 'required|email',
        'error' => 'Email appears to be invalid.',
    ],
    'password' => [
        'rules' =>'required|min:8',
        'error' => 'Your password must be atleast 8 characters long.',
    ]
]);
```

You can also specify custom error message individually.

```php
$validator->setRule('email', [
    'rules' => 'required|email',
    'error'=> 'Email appears to be invalid',
]);
```

### Custom error labels

By default the validator will produce error messages using data field name. But sometimes
you might need to customize the field label. For that pass a `label` key with custom label value.

For example, consider this form:

```php
<form method="post">
    <input name="fname">
</form>
```

To validate form data:

```php
$validator->setRule('fname', 'required');
```

If you echo the error message in case form field is empty, it will produce error message using
`fname` as label.

```php
echo $validator->getError('fname'); // Fname is required
```

This is because the validator tries to convert the field `fname` to human readable format. To make
things better, either you provide custom error message:

```php
$validaor->setRule('fname' =>[
    'rules' =>'required',
    'error' => 'Please provide you first name'
]);
```

Or to avoid setting custom message, simply specify a custom label:

```php
$validator->setRule('fname', [
    'rules' => 'required',
    'label' => 'First name'
]);
```

Now this will produce error message with provided label.

```php
echo $validator->getError('fname'); // First name is required
```

## Validating Nested Array

You can validate nested array data as well. For example imagine you have a form with a nested array:

```php
<form method="post">
    <input name="title">
    <input name="address[city]">
    <input name="address[state]">
    <input name="address[zip]">
</form>
```

When the form is submitted, `$_POST` may look like this:

```php
$_POST = [
    'title' => 'My Title',
    'address' => [
        'city' => 'My City',
        'state' => 'My State',
        'zip' => 'My Zip',
    ],
];
```

To validate this data, you can use `setRules()` method to set rules for each field using `dot` syntax.

```php
$validator->setRules([
    'title' => 'required|alpha',
    'address.city' => 'required',
    'address.state' => 'required',
    'address.zip' => 'required|numeric',
]);
```

## Callbacks as rules

There might be cases when you might want to set your own custom rules. You can do so using **callbacks** as rules.

Pass a `closure` and define your own rule logic that should return a `boolean` value.

```php
$validator->setRule('visibility', function($status) {
    return $status === 'active'; 
});
```

You can also explicitly specify the **callback** in `rules` key:

```php
$validator->setRule('visibility', [
    'rules' => function($status) { 
        return $status === 'active'; 
    }
]);
```

## Available Validation Rules

Lightpack provides a good number of validation rules for most frequently
used scenarios. Below is a list of available rules with example.

### required

This rule validates if a data field is not empty.

```php
$validator->setrule('password', 'required');
```

### alpha

This rule validates that a field has only ASCII characters `A-Z, a-z`.

```php
$validator->setrule('name', 'alpha');
```

### alnum

This rule validates that a field has only ASCII characters `A-Z, a-z, 0-9`.

```php
$validator->setrule('username', 'alnum');
```

### email

This rule check for a valid email.

```php
$validator->setrule('email', 'email');
```

### slug

This rule validates that a field has a valid slug text. That means any combination
of `A-Z, a-z, 0-9, -, _` characters is valid.

```php
$validator->setrule('id', 'slug');
```

### url

This rule validates that a field has a valid URL.

```php
$validator->setrule('website', 'url');
```

### ip

This rule validates that a field has a valid IP address.

```php
$validator->setrule('server', 'ip');
```

### length

This rule checks that a field has exactly `N` ASCII characters where `N` is an integer.

```php
$validator->setRule('phone', 'length:10');
```

### min

This rule checks whether a field has a minimum value provided as an integer.

```php
$validator->setRule('name', 'min:3');
```

### max

This rule checks whether a field exceeds maximum value provided as an integer.

```php
$validator->setRule('name', 'max:25');
```

### between

This rule checks if a field has a value between a minimum and maxium value.

```php
$validator->setRule('age', 'between:18,30');
```

**NOTE** The `between` filter also includes the range values. So the above example specifies
to validate the field where `18 <= age <= 30`.

### date

This rule check if a field has valid date format. You need to provide a valid
date format seperated by ':'.

```php
$validator->setRule('birthday', 'date:d-m-Y');
$validator->setRule('last_login', 'date:YYYY-MM-DD');
```

### before

This rule checks if a field has date value before a given date.

```php
$validator->setRule('registration_date', 'before:/d-m-Y,12-09-2021/');
```

Notice the syntax for providing before date filter. You must provide a valid
date format and a date value seperatedby a comma.

### after

This rule checks if a field has date value after a given date.

```php
$validator->setRule('last_payment', 'after:/d-m-Y,12-09-2020/');
```

Notice the syntax for providing before date filter. You must provide a valid
date format and a date value seperatedby a comma.

### match

This rule checks if a field has same value as with another field. For example, consider this form:

```php
<form method="post">
    <input type="password" name="password">
    <input type="password" name="confirm_password">
</form>
```

When the above is posted, we might want to validate that `confirm_password` field has same value 
as that of `password` field. Use the `matches` filter to do so.

```php
$validator->setRule('password', 'required|min:8|max:45');
$validator->setRule('confirm_password', 'match:password=' . $_POST['password']);
```

**NOTE:** You must pass the matching field name and its value seperated by `=` equals sign. 

### regex

Custom `regular expression` based validation logic is also supported.

```php
$validator->setRule('phone', 'regex:/^\d{3}-\d{3}-\d{4}$/');
```