# Lightpack Http Client

Lightpack provide a lightweight **HTTP client** for making API requests.

- **Fluent API:** Chain methods for configuration and requests.
- **Supports all major HTTP verbs:** GET, POST, PUT, PATCH, DELETE.
- **Flexible data formats:** JSON (default) and form-urlencoded.
- **Custom headers, bearer token, and file uploads.**
- **Timeouts, SSL options, and custom cURL settings.**
- **Simple helpers for response status, errors, and parsing.**

---

## Quick Reference

| Method                  | Purpose                                               |
|------------------------|------------------------------------------------------|
| `get($url, $query)`    | Send GET request with optional query params           |
| `post($url, $data)`    | Send POST request with data (JSON or form)            |
| `put($url, $data)`     | Send PUT request with data                            |
| `patch($url, $data)`   | Send PATCH request with data                          |
| `delete($url, $data)`  | Send DELETE request with data                         |
| `headers([$k=>$v])`    | Set custom headers                                    |
| `files([$k=>$file])`   | Attach files for upload (POST/PUT)                    |
| `token($token)`        | Set Bearer token for Authorization                    |
| `timeout($seconds)`    | Set request timeout                                   |
| `insecure()`           | Disable SSL verification (use with caution)           |
| `options([$opt=>$v])`  | Set custom cURL options                               |
| `form()`               | Send data as form-urlencoded                          |
| `download($url, $path)`| Download file to disk                                 |
| `json()`               | Get response as array (assumes JSON)                  |
| `body()`               | Get raw response body                                 |
| `status()`             | Get HTTP status code                                  |
| `error()`              | Get transport error message, if any                   |
| `ok()`                 | 2xx status                                            |
| `failed()`             | Transport or HTTP error                               |
| `clientError()`        | 4xx status                                            |
| `serverError()`        | 5xx status                                            |
| `redirect()`           | 3xx status                                            |

---

## Creating a Client

```php
use Lightpack\Http\Http;

$http = new Http();
```

---

## Making Requests

### GET Request
```php
$response = $http->get('https://api.example.com/users');
$data = $response->json();
```

### GET with Query Parameters
```php
$response = $http->get('https://api.example.com/users', [
    'name' => 'alice',
    'active' => true
]);
```

### POST Request (JSON)
```php
$response = $http->post('https://api.example.com/users', [
    'name' => 'alice',
    'email' => 'alice@example.com'
]);
```

### POST Request (Form Data)
```php
$response = $http
    ->form()
    ->post('https://api.example.com/login', [
        'username' => 'alice',
        'password' => 'secret',
    ]);
```

### PUT/PATCH/DELETE Requests
```php
$http->put($url, $data);
$http->patch($url, $data);
$http->delete($url, $data);
```

---

## Customizing Requests

### Custom Headers
```php
$headers = ['X-Custom' => 'value', 'Accept' => 'application/json'];

$response = $http->headers($headers)->get($url);
```

### Bearer Token Authentication
```php
$response = $http->token('your-token')->get($url);
```

### File Uploads
```php
$file = ['avatar' => '/path/to/file.jpg'];

$response = $http->files()->post($url);
```

### Set Timeout
```php
$response = $http->timeout(10)->get($url);
```

### Insecure Requests (Disable SSL Verification)
```php
$response = $http->insecure()->get($url);
```

### Custom cURL Options
```php
$curlOptions = [
    CURLOPT_PROXY => 'proxy.example.com:8080',
    CURLOPT_FOLLOWLOCATION => false
];

$response = $http->options()->get($url);
```

---

## Handling Responses

### Status and Error Checking
```php
if ($response->ok()) {
    // 2xx response
}
```

```php
if ($response->clientError()) {
    // 4xx error
}
```

```php
if ($response->serverError()) {
    // 5xx error
}
```

```php
if ($response->failed()) {
    // Transport error (DNS, connection, SSL, etc.)
    $error = $response->error();
}
```

### Parsing Response Data
```php
$array = $response->json(); // Array (assumes JSON)
$text = $response->body();  // Raw body
$status = $response->status(); // HTTP status code
```

---

## Downloading Files

```php
$success = $http->download('https://example.com/image.jpg', '/tmp/image.jpg');

if ($success) {
    // File downloaded
}
```

---

## Error Handling Explained

- **Transport Errors:**
  - Occur before an HTTP response is received (DNS failure, timeout, SSL error, etc.)
  - `$response->error()` returns the error message
  - `$response->status()` returns 0
  - `$response->failed()` returns true

- **HTTP Errors:**
  - Server returns 4xx or 5xx status
  - `$response->status()` returns the HTTP code
  - `$response->failed()` returns true

---

## Example: Complete Request Flow

```php
$http = new Http();

$response = $http
    ->token('abc123')
    ->headers(['Accept' => 'application/json'])
    ->timeout(10)
    ->get('https://api.example.com/profile');

if ($response->ok()) {
    $profile = $response->json();
} elseif ($response->failed()) {
    // Handle error
    $error = $response->error();
}
```

---