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

You can also chain calls to `setRule()` methods:

```php
$validator
    ->setRule('email', 'required|email');
    ->setRule('password', 'required|min:8');
```

### Setting rules together

To set rules for multiple fields together, call `setRules()` method:

```php
$validator->setRules([
    'email', 'required|email',
    'password', 'required|min:8',
];
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