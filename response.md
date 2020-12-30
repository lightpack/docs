# Response

The <code>Framework\Http\Response</code> class provides few utility methods to help with
generating appropriate response to output.

An instance of this class has already been configured in service container. So you can access
it anywhere using <code>app('response')</code> function.

```php
$response = app('response');
```

## Code

To set the response status code, use <code>setCode()</code> method.

```php
$reponse->setCode(404);
```

To get the set response code, use <code>getCode()</code> method.

```php
$rsponse->getCode();
```

<p class="tip">By default, the response code is set to <code>200</code>.</p>

## Type

To set the resonse content type, use <code>setType()</code> method.

```php
$reponse->setType('application/json');
```

To get the set reponse content type, use <code>getType()</code> method.

```php
$response->getType();
```

<p class="tip">By default, the response content type is set to <code>text/html</code></p>

## Message

To set the reponse status message, use <code>setMessage()</code> method.

```php
$response->setMessage('Not Found');
```

To get the set response status message, use <code>geMessage()</code> method.

```php
$response->getMessage();
```

<p class="tip">By default, the response status message is set to <code>OK</code></p>

## Headers

To set a response header, use <code>setHeader()</code> method.

```php
$response->setHeader('Server', 'Apache/2.4.1 (Unix)');
$response->setHeader('X-Frame-Options', 'SAMEORIGIN');
```

To set multiple response headers at once, use <code>setHeaders</code> method.

```php
$response->setHeaders([
    'Server' => 'Apache/2.4.1 (Unix)',
    'X-Frame-Options' => 'SAMEORIGIN'
]);
```

To get all the set response headers, use <code>getHeaders()</code> method.

```php
$response->getHeaders();
```

## Body

To set the response content body, use <code>setBody()</code> method.

```php
$response->setBody('Hello World');
```            

To get the set response body, use <code>getBody()</code> method.

```php
$response->getBody();
```        

## Send

After you have configured the appropriate response, you would want to send the output to
the client. For that, simply call <code>send()</code> method. It takes care of properly
formatting the response output for you without you having to worry about preparing the
output.

```php
$response->send();
```

## JSON

To send a <code>JSON</code> response, use the <code>json() method.</code> Pass it an array of data
to be sent as JSON response body. It takes care of setting content-type header to <code>application/json</code>.

```php
$response->json(['name' => 'Bob'])->send();
```

## XML

To send an <code>XML</code> response, use the <code>xml()</code> method. Pass it an
XML formatted string to send as response. It takes care of setting content-type header to <code>text/xml</code>.

```php
$response->xml('xml-string')->send();
```

## Chaining

Method chaining is supported to assist you in preparing the appropriate response.

```php
$response->setCode(200)->setMessage('OK')->setType('text/html')->setBody('Hello World')
```