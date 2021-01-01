# Request

The <code>Framework\Http\Request</code> class provides utility methods to deal with
incoming HTTP request. 

Lightpack already configures an instance of <code>Request</code> in IoC container.
So you can simply access it anywhere in your project using helper function <code>app()</code>.
```php
$request = app('request');
```

## URL

Assuming your application is installed in <code>app</code> folder within web root, if 
the request URL is <code>http://localhost/app/users/editors?status=active</code>, following
request methods might be of help for you.

```php
// Gives: /app/users/editors
$request->uri();

// Gives: /users/editors
$request->path();

// Gives: /app
$request->basepath();

// Gives: status=active
$request->query();

// Gives: /app/users/editors?status=active
$request->fullUri();
```

## Verbs

Following methods are available to access common HTTP verbs:

```php

// Return true if HTTP method is GET
$request->isGet(); 

// Return true if HTTP method is POST
$request->isPost();

// Return true if HTTP method is PUT
$request->isPut();

// Return true if HTTP method is PATCH
$request->isPatch();

// Return true if HTTP method is DELETE
$request->isDelete();
```

## Input

To access form input values submitted via HTTP <code>GET</code> or <code>POST</code>
request:

```php
$request->get('key');
$request->post('key');
```

Provide second parameter for default input value:

```php
$request->input('key', 'default');
```

Passing no key will return global $_GET/$_POST arrays.

```php
$request->get(); // returns $_GET
$request->post(); // returns $_POST
```

## AJAX

To check if the incoming request is an AJAX request:

```php
$request->isAjax();
```

> Note: This method relies on <code>X-Requested-With</code> header.

## JSON

To check if the incoming request has asked for JSON data:

```php
$request->isJson();
```

## Secure

To check if the incoming request is secure i.e. <code>https</code>:

```php
$request->isSecure();
```
