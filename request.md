# Request

The <code>Lightpack\Http\Request</code> class provides utility methods to deal with
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
$request->get('key', 'default');
$request->post('key', 'default');
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

## Uploads

Consider this file upload form:

```html
<form method="post" enctype="multipart/form-data">
    <input name="photo" type="file" />
    <button>Upload</button>
</form>
```

### Retrieve uploaded file

To retrieve the uploaded file details:

```php
$file = app('request')->file('photo');
```

### Save uploaded file

To **save/move** the uploaded file locally:

```php
$file->move(DIR_STORAGE . '/uploads');
```

You can also pass the filename as second argument:

```php
$file->move(DIR_STORAGE . '/uploads', 'filename.jpg');
```

### Uploaded file name

To get the uploaded filename:

```php
$file->getName();
```

### Uploaded file size

To get the uploaded filesize in bytes:

```php
$file->getSize();
```

### Uploaded file type

To get the uploaded file type:

```php
$file->getType();
```

### Uploaded file extension

To get the uploaded file extension:

```php
$file->getExtension();
```

### Uploaded file error

To check if there was an error while uploading file:

```php
$file->hasErrors(); // true|false
```

To get the file upload error:

```php
$file->getError();
```

### Multiple File Uploads

Consider this multiple file upload form:

```html
<form method="post" enctype="multipart/form-data">
    <input name="photos[]" type="file" multiple/>
    <button>Upload</button>
</form>
```

To retrieve all the uploaded photos:

```php
$files = app('request')->files('photos');
```

Now all of the above mentioned file methods can be applied individually on each uploaded file. For example, here we loop each uploaded photo and store them:

```php
foreach($files as $file) {
    $file->move(DIR_STORAGE . '/uploads');
}
```