# Data Validation

When accepting data from a source such as form input, you may want to validate data
itself to ensure you get the data as expected. `Lightpack` gives you a simple approach
to validate data for most of the use cases. 

<p class="tip">While you are free to use any data validation library of your choice, Lightpack provides <code>Lightpack\Validator\Validator</code> class for data validation which you might find handy enough
for your data validation needs.</p>

## Quick example

First create an instance of `Lightpack\Validator\Validator` passing it the data source.

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