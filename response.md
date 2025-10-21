# Response

The `Lightpack\Http\Response` class provides a comprehensive, fluent API for generating HTTP responses in your application. It supports status codes, headers, content types, file downloads, streaming, security, caching, advanced output, and more.

An instance is automatically available via the `response()` helper.

---

## Status, Type, Message, and Body

- **Set Status Code:**
  ```php
  response()->setStatus(404);
  ```
  - Sets both the code and the standard status message, unless overridden.
  - Default: `200` (OK)

- **Get Status Code:**
  ```php
  $code = response()->getStatus();
  ```

- **Set/Override Status Message:**
  ```php
  response()->setMessage('Not Found');
  ```
  - Default: Standard HTTP message for the code (e.g., `OK` for 200).

- **Get Status Message:**
  ```php
  $msg = response()->getMessage();
  ```

- **Set Content Type:**
  ```php
  response()->setType('application/json');
  ```
  - Default: `text/html`
  - Charset is always set to UTF-8.

- **Get Content Type:**
  ```php
  $type = response()->getType();
  ```

- **Set Response Body:**
  ```php
  response()->setBody('Hello World');
  ```

- **Get Response Body:**
  ```php
  $body = response()->getBody();
  ```

---

## Header Management

- **Set a Header:**
  ```php
  response()->setHeader('X-Frame-Options', 'SAMEORIGIN');
  ```

- **Set Multiple Headers:**
  ```php
  response()->setHeaders([
      'Server' => 'Apache',
      'X-Frame-Options' => 'SAMEORIGIN',
  ]);
  ```

- **Get All Headers:**
  ```php
  $headers = response()->getHeaders();
  ```

- **Get a Header by Name:**
  ```php
  $value = response()->getHeader('X-Frame-Options');
  ```

- **Check if Header Exists:**
  ```php
  if (response()->hasHeader('Content-Type')) { ... }
  ```

---

## Sending the Response

- **Send the response to the client:**
  ```php
  response()->send();
  ```
---


## Method Chaining

All setters return `$this` for fluent chaining:

```php
response()
    ->setStatus(201)
    ->setType('application/json')
    ->setHeader('X-Resource', 'created')
    ->json(['id' => 5])
    ->send();
```

---

## JSON, XML, and Plain Text Responses

- **JSON Response:**
  ```php
  response()->json(['success' => true]);
  ```
  - Sets Content-Type to `application/json` and encodes the data.
  - Throws exception if encoding fails.

- **XML Response:**
  ```php
  response()->xml('<root><name>Bob</name></root>');
  ```
  - Sets Content-Type to `text/xml`.

- **Plain Text Response:**
  ```php
  response()->text('Hello World');
  ```
  - Sets Content-Type to `text/plain`.

---

## File Downloads and Streaming

- **Download a locally stored file:**
  ```php
  response()->download('/path/to/file.pdf');
  response()->download('/path/to/file.pdf', 'custom.pdf');
  response()->download('/path/to/file.pdf', null, ['X-Header' => 'Value']);
  ```

  - Sets appropriate headers for download (`Content-Disposition: attachment`).
  - Automatically detects MIME type via `MimeTypes::getMime()`.
  - Uses `file_get_contents()` to read entire file into memory.
  - **Not recommended for files larger than a few MB** - use `downloadStream()` instead.
  - Does not throw exceptions - ensure file exists before calling.

- **Stream a File Download (Memory Efficient):**
  ```php
  response()->downloadStream('/path/to/large.zip');
  
  // Stream in 2MB chunks
  response()->downloadStream('/path/to/large.zip', 'archive.zip', [], 2*1024*1024); 
  ```

  - Streams file in chunks to client (default: 1MB chunks).
  - **Throws `RuntimeException`** if file does not exist.
  - Disables output buffering with `ob_end_clean()`.
  - Checks `connection_status()` to stop if client disconnects.
  - Essential for files larger than a few MB.

- **Display a File Inline in Browser:**
  ```php
  response()->file('/path/to/image.jpg');
  response()->file('/path/to/image.jpg', 'photo.jpg');
  response()->file('/path/to/image.jpg', null, ['X-Custom' => 'Value']);
  ```

  - Reads entire file into memory - not for large files.

- **Stream a File Inline:**
  ```php
  response()->fileStream('/path/to/large-video.mp4');
  response()->fileStream('/path/to/large-video.mp4', 'video.mp4', [], 2*1024*1024);
  ```
  
  - **Throws `RuntimeException`** if file does not exist.
  - Ideal for streaming videos, PDFs, or large images.

- **Best Practice:** Always use streaming methods for files larger than a few megabytes.

---


## Streaming Responses

Lightpack’s `Response` class provides several ways to stream data to the client in a memory-efficient, real-time manner. This is essential for large files, live data, CSV exports, and server-sent events (SSE).

### 1. Arbitrary Content Streaming (`stream()`)

Use `stream()` to send custom output in real time, such as:
- Large dynamically-generated content
- Server-Sent Events (SSE)
- Progress updates
- Long-running reports

**Example: Real-time progress output**
```php
response()->stream(function() {
    for ($i = 1; $i <= 10; $i++) {
        echo "Progress: $i/10\n";
        flush(); // Push output to client immediately
        sleep(1);
    }
});
```

**Example: Server-Sent Events (SSE)**
```php
response()
    ->setHeader('Content-Type', 'text/event-stream')
    ->stream(function() {
        for ($i = 0; $i < 5; $i++) {
            echo "data: Message $i\n\n";
            ob_flush(); flush();
            sleep(2);
        }
    });
```

**getStreamCallback():**
- Use `getStreamCallback()` to retrieve the currently set stream callback function.
- This is useful for testing, introspection, or advanced middleware that needs to inspect or manipulate the streaming logic before sending the response.

**Example: Inspecting the stream callback**
```php
$response = response()->stream(function() {
    echo 'Streaming...';
});

$callback = $response->getStreamCallback();
if (is_callable($callback)) {
    // You can inspect, wrap, or call the callback as needed
}
```

**Notes:**
- Always call `flush()` or `ob_flush()` to ensure data is sent immediately.
- Use `setHeader()` to set appropriate content types (e.g., `text/event-stream` for SSE).
- The callback receives no arguments; use closures to capture needed variables.
- Streaming disables the normal response body (`setBody()` is ignored if a stream is set).

### 2. File Streaming for Downloads (`downloadStream()`)

Use `downloadStream()` for memory-efficient, chunked file downloads:

**Example: Download a large file in 2MB chunks**
```php
response()->downloadStream('/path/to/huge.zip', 'backup.zip', [], 2 * 1024 * 1024);
```
- Automatically sets headers for file download.
- Reads and outputs the file in chunks (default 1MB, configurable).
- Avoids loading the entire file into memory.
- Throws an exception if the file does not exist.

**Best Practice:** Use for files larger than a few megabytes.

### 3. File Streaming for Inline Display (`fileStream()`)

Use `fileStream()` to stream large files for direct display in the browser (e.g., videos, PDFs):

**Example: Stream a video inline**
```php
response()->fileStream('/media/bigvideo.mp4');
```
- Sets `Content-Disposition: inline` so the browser displays the file.
- Uses chunked output for memory efficiency.

### 4. CSV Streaming (`streamCsv()`)

Use `streamCsv()` to efficiently export large datasets as CSV downloads:

**Example: Export users as CSV**
```php
response()->streamCsv(function() {
    echo "id,name\n";
    foreach ($users as $user) {
        echo $user->id . ',' . $user->name . "\n";
    }
}, 'users.csv');
```

**What it does:**
- Sets `Content-Type: text/csv`
- Sets `Content-Disposition: attachment; filename="users.csv"`
- The callback writes CSV rows directly to output
- Ideal for exporting large or streaming datasets

**Default filename:** If you don't provide a filename, it defaults to `'export.csv'`.

### 5. Streaming Gotchas and Best Practices

- **Headers:** All headers must be set before returning the response. Once output starts, headers cannot be changed.
- **Buffering:** Use `flush()` or `ob_flush()` to push output.
- **Connection Handling:** If the client disconnects, the stream callback should handle it (see `connection_status()` in PHP).
- **Error Handling:** For file streams, exceptions are thrown if the file is missing. For custom streams, handle errors in your callback.
- **Memory Usage:** Streaming is essential for large files or datasets—never read large files into memory with `setBody()` or `download()`.
- **Chaining:** All streaming methods are chainable with other response configuration methods.

---

## Working with Redirects

### What is an HTTP Redirect?
An HTTP redirect tells the browser to immediately load a different URL. This is done by sending a special status code (usually 302) and a `Location` header in the response. Redirects are commonly used after form submissions, login/logout, or when a resource has moved.

In Lightpack, redirects are handled by the `Redirect` class, accessible via the `redirect()` helper. This provides a clear, expressive, and testable API for all common redirect scenarios.

---

### Common Redirect Methods

#### **redirect()->to($url)**
Redirect to any absolute or relative URL.

**Example: After login, send user to dashboard**
```php
public function login()
{
    // ... authentication logic ...
    return redirect()->to('/dashboard');
}
```
- Sets status code 302 and `Location: /dashboard` header.
- Works with both absolute and relative URLs.

#### **redirect()->route($name, ...$params)**
Redirect to a named route, passing route parameters as needed.

**Example: After profile update, redirect to profile page**
```php
public function update($userId)
{
    // ... update logic ...
    return redirect()->route('profile', $userId);
}
```
- Resolves the route name and parameters to a URL.
- Ensures your redirects stay in sync with route changes.

#### **redirect()->back()**
Redirect back to the previous page (using URL stored in session, or `/` if unavailable).

**Example: After failed form validation, return user to previous page**
```php
public function save()
{
    if (! $this->validate()) {
        // ... maybe set flash message ...
        return redirect()->back();
    }
    // ...
}
```
- Uses session to remember the last visited URL.
- Essential for post/redirect/get patterns.

#### **redirect()->intended()**
Redirect to the “intended” URL stored in session (commonly used after login), or `/` if not set.

**Example: After login, send user to originally requested page**
```php
public function login()
{
    // ... authentication logic ...
    return redirect()->intended();
}
```
- Stores the intended URL before redirecting unauthenticated users.
- After login, users are returned to where they originally wanted to go.

#### **redirect()->refresh()**
Redirect to the current URL (refresh the page).

**Example: After a POST, refresh the page to show updated data**
```php
public function post()
{
    // ... save logic ...
    return redirect()->refresh();
}
```

**How it works:**
- Uses `request()->fullUrl()` to get the current full URL (including query string).
- Sets status 302 and `Location` header to current URL.
- Useful for preventing duplicate form submissions via POST/Redirect/GET pattern.

---

### Common Patterns and Pitfalls
- **Always return the redirect from your controller:**
  ```php
  return redirect()->to('/login');
  ```
- **Chain additional headers or configuration:**
  ```php
  return redirect()->to('/login')->setHeader('X-Reason', 'auth-required');
  ```
- **Flash messages:** Set flash data before redirecting to show messages after redirect.
- **Session integration:** `back()` and `intended()` rely on session data—ensure sessions are enabled.
- **Prefer `redirect()` over manual Location headers:** It’s safer, more expressive, and integrates with Lightpack’s routing/session system.
- **Advanced:** For custom needs, use `setRedirectUrl()` and `getRedirectUrl()` on any response, but the `Redirect` class covers all standard use cases.

---

## View Rendering

- **Render a Template:**
  ```php
  response()->view('emails/welcome', ['name' => 'Alice']);
  ```
  - Renders an HTML template as the response body.
  - The second parameter allows passing an array of data to be available to view templates.
  - Each array data key becomes a variable available to the view file.

---

## Security Headers

- **Apply Secure Defaults:**
  ```php
  response()->secure();
  ```
  - Adds recommended security headers:
    - `X-Content-Type-Options: nosniff`
    - `X-Frame-Options: SAMEORIGIN`
    - `X-XSS-Protection: 1; mode=block`
    - `Referrer-Policy: strict-origin-when-cross-origin`
    - Adds HSTS if HTTPS is detected.
  - You can also pass custom headers to override or add:
    ```php
    response()->secure(['X-Frame-Options' => 'DENY']);
    ```

---

## Caching and Last-Modified

- **Enable HTTP Caching:**
  ```php
  response()->cache(3600); // 1 hour
  response()->cache(3600, ['public' => false, 'immutable' => true]);
  ```
  - Sets `Cache-Control`, `Expires`, and `Pragma` headers.
  - `public` (default true): Allow shared caches; `private`: only for user.
  - `immutable`: Content won’t change during cache lifetime.

- **Disable Caching:**
  ```php
  response()->noCache();
  ```
  - Sets appropriate headers to prevent all caching.

- **Set Last-Modified Header:**
  ```php
  response()->setLastModified(time());
  response()->setLastModified('2024-07-17 12:00:00');
  response()->setLastModified(new DateTime());
  ```
  - Accepts timestamp, date string, or DateTime object.

---

## Summary Table of Methods

| Method                       | Purpose                                      |
|------------------------------|----------------------------------------------|
| setStatus(int)               | Set HTTP status code                         |
| getStatus()                  | Get HTTP status code                         |
| setMessage(string)           | Set HTTP status message                      |
| getMessage()                 | Get HTTP status message                      |
| setType(string)              | Set Content-Type                             |
| getType()                    | Get Content-Type                             |
| setHeader(name, value)       | Set a header                                 |
| setHeaders(array)            | Set multiple headers                         |
| getHeaders()                 | Get all headers                              |
| getHeader(name)              | Get a header value                           |
| hasHeader(name)              | Check if header is set                       |
| setBody(string)              | Set response body                            |
| getBody()                    | Get response body                            |
| send()                       | Send headers and body                        |
| setTestMode(bool)            | Prevent exit() (for tests)                   |
| json(data)                   | Set JSON response                            |
| xml(string)                  | Set XML response                             |
| text(string)                 | Set plain text response                      |
| download(path, ...)          | Download a file                              |
| downloadStream(path, ...)    | Stream file download (large files)           |
| file(path, ...)              | Display a file inline                        |
| fileStream(path, ...)        | Stream file inline (large files)             |
| stream(callable)             | Stream custom output                         |
| getStreamCallback()          | Get stream callback                          |
| view(file, data)             | Render and set a template                    |
| streamCsv(callable, name)    | Stream CSV file for download                 |
| secure([headers])            | Add security headers                         |
| cache(maxAge, [options])     | Enable HTTP caching                          |
| noCache()                    | Disable HTTP caching                         |
| setLastModified(time)        | Set Last-Modified header                     |
| setRedirectUrl(url)          | Set a logical redirect URL                   |
| getRedirectUrl()             | Get the logical redirect URL                 |

---