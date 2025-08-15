# Request

The `Lightpack\Http\Request` class provides a comprehensive API for accessing and interacting with incoming HTTP requests.

Lightpack automatically configures an instance of `Request` in the IoC container. You can access it anywhere in your project using the `request()` helper.

---

## URL and Path Methods

Assuming your application is installed in the `app` folder within the web root, and the request URL is `http://localhost/app/users/editors?status=active`, the following methods are available:

```php
// Returns: /app/users/editors?status=active
request()->uri();

// Returns: /app/users/editors
request()->fullpath();

// Returns: /users/editors
request()->path();

// Returns: /app
request()->basepath();

// Returns: http://localhost/app/users/editors
request()->url();

// Returns: http://localhost/app/users/editors?status=active
request()->fullUrl();
```

### Path Segments

```php
// Returns: ['users', 'editors']
request()->segments();

// Returns: 'users'
request()->segments(0);

// Returns: 'editors'
request()->segments(1);
```

---

## HTTP Methods and Verbs

```php
// Returns the HTTP method as a string (e.g., 'GET', 'POST', etc.)
request()->method();

// Returns true if HTTP method is GET
request()->isGet();

// Returns true if HTTP method is POST
request()->isPost();

// Returns true if HTTP method is PUT
request()->isPut();

// Returns true if HTTP method is PATCH
request()->isPatch();

// Returns true if HTTP method is DELETE
request()->isDelete();

// Returns true if the HTTP method was spoofed via _method
request()->isSpoofed();

// Returns an array of all supported HTTP verbs
request()->verbs();
```

> **Note:** Method spoofing allows you to use PUT, PATCH, or DELETE via a POST request with a hidden `_method` field.

---

## Input and Query Data

### Merged Input

The `input()` method intelligently merges input from query string, POST data, JSON body, or parsed body depending on the HTTP method and content type.

```php
// Get a specific input value
request()->input('key');

// Get a specific input value with a default
request()->input('key', 'default');

// Get all input data as an array
request()->input();
```

### Query String

```php
// Get a specific query parameter
request()->query('status');

// Get all query parameters as an array
request()->query();
```

### Raw and Parsed Body

```php
// Get the raw request body as a string
request()->getRawBody();

// Get the parsed body (for PUT/PATCH/DELETE with form data)
request()->getParsedBody();

// Get a specific parsed body value with a default
request()->getParsedBody('key', 'default');
```

---

## JSON Requests

```php
// Returns true if the request Content-Type is application/json
request()->isJson();

// Returns true if the client expects a JSON response (Accept: application/json)
request()->expectsJson();
```

> **Deprecated:** `json()` is an alias for `input()` and is deprecated. Use `input()` instead.

---

## AJAX Requests

```php
// Returns true if the request was made via AJAX (X-Requested-With: XMLHttpRequest)
request()->isAjax();
```

---

## Security and Protocol

```php
// Returns true if the request is over HTTPS
request()->isSecure();

// Returns the scheme (http or https)
request()->scheme();

// Returns the HTTP protocol version (e.g., 'HTTP/1.1')
request()->protocol();

// Returns the Content-Type of the request
request()->format();
```

---

## Host and Port

```php
// Returns the host name (without port)
request()->host();

// Returns the port number (if available)
request()->port();

// Returns the host with port (e.g., 'localhost:8080')
request()->hostWithPort();
```

---

## Headers

```php
// Get a specific header value
request()->header('User-Agent');

// Get all headers as an associative array
request()->headers();

// Check if a header exists
request()->hasHeader('Authorization');

// Get the User-Agent string
request()->userAgent();

// Get the Bearer token from the Authorization header
request()->bearerToken();
```

---

## CSRF Token

```php
// Get the CSRF token from header or input
request()->csrfToken();
```

---

## Referer

```php
// Get the referer URL, if present
request()->referer();
```

---

## Route Information

```php
// Get the matched route instance (if available)
request()->route();

// Get all route parameters as an array
request()->params(null);

// Get a specific route parameter (with optional default)
request()->params('userId', 0);
```

To check if the current request's resolved route name matches the given pattern or array of patterns. Wildcard patterns are supported, allowing flexible route checks for authorization, navigation, or context-aware logic.

```php
// Returns true if the current route's name is 'dashboard'
request()->matchesRoute('dashboard');

// Returns true if the current route's name starts with 'admin.'
request()->matchesRoute('admin.*');

// Returns true if the current route's name matches any of the patterns
request()->matchesRoute(['dashboard', 'admin.*']);
```

---

## URL Signature Validation

```php
// Throws an exception if the URL signature is invalid
request()->validateUrlSignature();

// Returns true if the URL signature is valid
request()->hasValidSignature();

// Returns true if the URL signature is invalid
request()->hasInValidSignature();
```

## Client IP Address

```php
// Get the client's IP address
request()->ip();
```

## File Uploads

Lightpack provides a robust API for handling file uploads via the `request()->file()` and `request()->hasFile()` methods. 

Uploaded file are represented by the `UploadedFile` class.

### Single File

```html
<input name="photo">
```

```php
// Retrieve the uploaded file by key
$file = request()->file('photo');

// Check if a file was uploaded for the given key
request()->hasFile('photo');
```

```php
$file = request()->file('avatar');

// Check if file upload failed
if ($file->hasError()) {
    //
}
```

### Multiple Files

```html
<input name="photos[]" type="file" multiple>
```

```php
$photos = request()->file('photos'); 

foreach ($photos as $photo) {
    if (!$photo->hasError()) {
        // Handle each uploaded file
    }
}
```

### UploadedFile Methods

Each `UploadedFile` instance provides:

- `getName()`: Original filename
- `getSize()`: File size in bytes
- `getType()`: MIME type (e.g., `image/jpeg`)
- `getExtension()`: File extension (e.g., `jpg`)
- `getTmpName()`: Temporary path on disk
- `hasError()`: Returns `true` if there was an upload error
- `getError()`: Returns the error code (see PHP file upload errors)
- `isEmpty()`: Returns `true` if file is empty
- `isImage()`: Returns `true` if file is an image
- `getDimensions()`: Returns `['width' => ..., 'height' => ...]` for images
- `getWidth()`, `getHeight()`: Image dimensions (0 if not an image)

### Storing Uploaded Files

Lightpack abstracts the storage of uploaded files to utilize the configured [Storage](storage.md) service.

#### Store in Arbitrary Location

```php
$path = $file->store('some/directory');
```

#### Store in Public Uploads

```php
$path = $file->storePublic('avatars');
```
- Stores in `uploads/public/avatars/` (web-accessible).

#### Store in Private Uploads

```php
$path = $file->storePrivate('documents');
```
- Stores in `uploads/private/documents/` (not web-accessible by default).

#### Storage Options

You can customize storage via the `$options` array:

- `'name'`: Custom filename (string or callable)
- `'unique'`: Generate a unique filename (bool)
- `'preserve_name'`: Keep original name as prefix when unique (bool)

Example:

```php
$path = $file->storePublic('avatars', [
    'unique' => true,
    'preserve_name' => true,
]);
```

### Troubleshooting

- `request()->file('key')` returns `null` if no file was uploaded.
- For multiple files, always check `is_array()` before looping.
- Always check `hasError()` to avoid processing failed uploads.
- Use `storePublic` for files that should be accessible via URL, and `storePrivate` for protected files.

---

## HTTP Headers

Lightpack provides a clean API for accessing request headers through the `request()` helper.

### Retrieving a Single Header

```php
// Get a specific header value (case-insensitive)
$userAgent = request()->header('User-Agent');
$auth = request()->header('Authorization');
$custom = request()->header('X-Custom-Header', 'default-value');
```

- Header names are case-insensitive and normalized internally.
- If the header is not present, the default value is returned (or `null` if not specified).

### Checking for a Header

```php
if (request()->hasHeader('X-Requested-With')) {
    // Handle AJAX request
}
```

### Retrieving All Headers

```php
$headers = request()->headers(); // Returns an associative array of all headers
foreach ($headers as $name => $value) {
    // Process each header
}
```

### Common Header Patterns

#### Authorization / Bearer Token

```php
$token = request()->bearerToken();
if ($token) {
    // Use the Bearer token for authentication
}
```

#### CSRF Token

```php
$csrf = request()->csrfToken();
```

#### User-Agent

```php
$ua = request()->userAgent();
```

#### Forwarded Headers

Lightpack automatically parses common proxy headers:

- `X-Forwarded-For` (client IP)
- `X-Forwarded-Proto` (scheme)
- `X-Forwarded-Host` (host)
- `X-Forwarded-Port` (port)

These are used internally by `request()->ip()`, `request()->scheme()`, etc., but you can also access them directly:

```php
$proto = request()->header('X-Forwarded-Proto');
```

### Notes

- All header methods are case-insensitive.
- For custom headers, always use the normalized name (e.g., `X-Custom-Header`).

---